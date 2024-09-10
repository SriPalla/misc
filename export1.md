Here's an optimized code example for a Spring Boot Kotlin Cloud Run app that exposes an API endpoint to handle a CloudEvent, routes it to the appropriate Pub/Sub topic based on message type from `application.yml`, and ensures publishers are reused per topic using coroutines.

### `application.yml`

```yaml
pubsub:
  topics:
    - eventType: "com.example.type1"
      topic: "projects/your-project-id/topics/topic1"
    - eventType: "com.example.type2"
      topic: "projects/your-project-id/topics/topic2"
```

### Pub/Sub Configuration (Singleton for Publisher)

```kotlin
import com.google.cloud.pubsub.v1.Publisher
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import java.util.concurrent.ConcurrentHashMap

@Configuration
class PubSubConfig {

    private val publishers: MutableMap<String, Publisher> = ConcurrentHashMap()

    @Bean
    fun getPublisherFactory(): (String) -> Publisher {
        return { topicName ->
            publishers.computeIfAbsent(topicName) {
                Publisher.newBuilder(topicName).build()
            }
        }
    }
}
```

### CloudEvent Processing Service

```kotlin
import com.google.cloud.pubsub.v1.Publisher
import com.google.protobuf.ByteString
import com.google.pubsub.v1.PubsubMessage
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import org.springframework.beans.factory.annotation.Value
import org.springframework.stereotype.Service

@Service
class CloudEventService(
    private val getPublisherFactory: (String) -> Publisher
) {

    @Value("\${pubsub.topics}")
    lateinit var topicMappings: List<TopicMapping>

    data class TopicMapping(val eventType: String, val topic: String)

    suspend fun publishCloudEvent(eventType: String, eventPayload: String) {
        val topic = topicMappings.find { it.eventType == eventType }?.topic
            ?: throw IllegalArgumentException("No topic found for event type $eventType")

        val publisher = getPublisherFactory(topic)
        val message = PubsubMessage.newBuilder()
            .setData(ByteString.copyFromUtf8(eventPayload))
            .build()

        withContext(Dispatchers.IO) {
            publisher.publish(message).get()
        }
    }
}
```

### Controller

```kotlin
import kotlinx.coroutines.runBlocking
import org.springframework.http.HttpStatus
import org.springframework.http.ResponseEntity
import org.springframework.web.bind.annotation.*

@RestController
@RequestMapping("/events")
class CloudEventController(
    private val cloudEventService: CloudEventService
) {

    @PostMapping
    fun handleCloudEvent(
        @RequestBody cloudEvent: CloudEventRequest
    ): ResponseEntity<String> = runBlocking {
        cloudEventService.publishCloudEvent(cloudEvent.type, cloudEvent.data)
        ResponseEntity.status(HttpStatus.ACCEPTED).body("Event processed")
    }
}

data class CloudEventRequest(val type: String, val data: String)
```

### Explanation

1. **Publisher Factory**: The `getPublisherFactory` bean ensures a new Pub/Sub `Publisher` is created for each topic and cached for reuse.
2. **Coroutine-Based Publishing**: The `publishCloudEvent` function runs the `publish` call inside a coroutine with `Dispatchers.IO` for non-blocking execution.
3. **Configuration Driven**: The `application.yml` stores the mappings between CloudEvent types and Pub/Sub topics. Adding a new mapping requires only a config update.
4. **10 TPS Support**: The reuse of publishers and non-blocking I/O using coroutines optimizes for performance, making 10 TPS achievable.

This setup is lightweight and scalable while ensuring that publishers are reused across requests.
