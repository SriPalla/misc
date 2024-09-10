Here’s an optimized approach to implement your Spring Boot Kotlin Cloud Run application without coroutines, supporting 10 TPS. The goal is to map each CloudEvent type to a specific Pub/Sub topic based on `application.yml` and reuse Pub/Sub publishers efficiently:

### 1. Define the Configuration in `application.yml`
```yaml
cloudEvent:
  mappings:
    - eventType: "event.type.1"
      pubsubTopic: "topic-1"
    - eventType: "event.type.2"
      pubsubTopic: "topic-2"
```

### 2. Create a Configuration Class to Read the Mappings
```kotlin
import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.context.annotation.Configuration

@Configuration
@ConfigurationProperties(prefix = "cloudEvent")
class CloudEventConfig {
    lateinit var mappings: List<CloudEventMapping>

    data class CloudEventMapping(
        var eventType: String = "",
        var pubsubTopic: String = ""
    )
}
```

### 3. Pub/Sub Publisher Management
Ensure each topic has a single, reusable publisher.
```kotlin
import com.google.cloud.pubsub.v1.Publisher
import com.google.pubsub.v1.TopicName
import org.springframework.stereotype.Service
import java.util.concurrent.ConcurrentHashMap

@Service
class PubSubPublisherManager {

    private val publishers = ConcurrentHashMap<String, Publisher>()

    fun getOrCreatePublisher(topicName: String): Publisher {
        return publishers.computeIfAbsent(topicName) {
            val topic = TopicName.of("your-project-id", topicName)
            Publisher.newBuilder(topic).build()
        }
    }

    fun shutdownAll() {
        publishers.values.forEach { it.shutdown() }
    }
}
```

### 4. CloudEvent to Pub/Sub Service
Map the CloudEvent to the corresponding topic and publish the event.
```kotlin
import com.google.protobuf.ByteString
import com.google.pubsub.v1.PubsubMessage
import org.springframework.stereotype.Service

@Service
class CloudEventPubSubService(
    private val cloudEventConfig: CloudEventConfig,
    private val publisherManager: PubSubPublisherManager
) {

    fun publishCloudEvent(cloudEventType: String, cloudEventData: String) {
        val mapping = cloudEventConfig.mappings
            .find { it.eventType == cloudEventType }
            ?: throw IllegalArgumentException("No mapping found for CloudEvent type: $cloudEventType")

        val publisher = publisherManager.getOrCreatePublisher(mapping.pubsubTopic)

        val message = PubsubMessage.newBuilder()
            .setData(ByteString.copyFromUtf8(cloudEventData))
            .build()

        publisher.publish(message)
    }
}
```

### 5. API Endpoint for Receiving CloudEvents
This exposes the API to receive CloudEvents, determine the event type, and publish to the appropriate Pub/Sub topic.
```kotlin
import org.springframework.web.bind.annotation.*

@RestController
@RequestMapping("/api/cloudEvent")
class CloudEventController(
    private val cloudEventPubSubService: CloudEventPubSubService
) {

    @PostMapping
    fun handleCloudEvent(@RequestBody cloudEvent: CloudEventRequest) {
        cloudEventPubSubService.publishCloudEvent(cloudEvent.type, cloudEvent.data)
    }

    data class CloudEventRequest(
        val type: String,
        val data: String
    )
}
```

### Key Points
- **Publisher Reuse**: Publishers are created once per topic and stored in a `ConcurrentHashMap`, avoiding the overhead of creating a new publisher for every message.
- **Configuration Flexibility**: Adding new event types and topics is as simple as updating the `application.yml`.
- **Thread Safety**: Using `ConcurrentHashMap` ensures thread-safe access to publishers.
- **Performance**: The design should handle 10 TPS without performance issues, as the publisher creation overhead is avoided after the first message for a topic.

This solution should be scalable and optimized for your use case without using coroutines.
