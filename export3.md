To create a Spring Boot Kotlin application on Cloud Run that handles CloudEvents and publishes them to appropriate Pub/Sub topics based on the event type, you can follow the steps below. We'll optimize the solution to support 10 TPS (transactions per second), reuse Pub/Sub publishers for topics, and configure the topic-event mappings through `application.yml`. For authentication between Cloud Run and Pub/Sub, we'll use app-to-app authentication leveraging Google Cloud IAM with appropriate permissions.

### Steps

1. **Configuration in `application.yml`:**
   Define the mapping of Pub/Sub topics to CloudEvent message types in `application.yml`:

```yaml
cloud:
  pubsub:
    mappings:
      "topicA": 
        - "eventTypeA1"
        - "eventTypeA2"
      "topicB": 
        - "eventTypeB1"
        - "eventTypeB2"
```

This structure defines a `Map<String, List<String>>` where the key is the Pub/Sub topic name, and the value is a list of CloudEvent types associated with that topic.

2. **CloudEvent Data Class:**
   Define the data class to model the CloudEvent:

```kotlin
data class CloudEvent(
    val id: String,
    val type: String,
    val source: String,
    val data: Map<String, Any>
)
```

3. **Configuration Class:**
   Read and store the Pub/Sub mappings from `application.yml` in a `Map<String, List<String>>`:

```kotlin
import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.context.annotation.Configuration

@Configuration
@ConfigurationProperties(prefix = "cloud.pubsub")
class PubSubConfig {
    lateinit var mappings: Map<String, List<String>>
}
```

4. **Pub/Sub Publisher Manager:**
   Create a singleton class that manages the Pub/Sub publishers, ensuring that each topic has only one publisher, which is reused across requests:

```kotlin
import com.google.cloud.pubsub.v1.Publisher
import com.google.pubsub.v1.TopicName
import org.springframework.stereotype.Component

@Component
class PubSubPublisherManager {

    private val publishers = mutableMapOf<String, Publisher>()

    fun getPublisher(topic: String): Publisher {
        return publishers.getOrPut(topic) {
            Publisher.newBuilder(TopicName.of("your-project-id", topic)).build()
        }
    }

    fun shutdown() {
        publishers.values.forEach { it.shutdown() }
    }
}
```

5. **Cloud Run Endpoint Controller:**
   Create an API endpoint that accepts CloudEvents, determines the correct topic based on the event type, and publishes the message to the topic. The publisher for each topic is reused:

```kotlin
import com.google.protobuf.ByteString
import com.google.pubsub.v1.PubsubMessage
import kotlinx.coroutines.runBlocking
import org.springframework.web.bind.annotation.*
import java.util.concurrent.TimeUnit

@RestController
@RequestMapping("/events")
class CloudEventController(
    private val pubSubConfig: PubSubConfig,
    private val publisherManager: PubSubPublisherManager
) {

    @PostMapping
    suspend fun handleCloudEvent(@RequestBody cloudEvent: CloudEvent) {
        val topic = findTopicForEventType(cloudEvent.type)
        val publisher = publisherManager.getPublisher(topic)

        val message = PubsubMessage.newBuilder()
            .setData(ByteString.copyFromUtf8(cloudEvent.data.toString()))
            .putAttributes("eventId", cloudEvent.id)
            .putAttributes("eventType", cloudEvent.type)
            .build()

        // Publish message asynchronously
        runBlocking {
            val messageIdFuture = publisher.publish(message)
            messageIdFuture.get(10, TimeUnit.SECONDS) // Ensure timely processing
        }
    }

    private fun findTopicForEventType(eventType: String): String {
        return pubSubConfig.mappings.entries.find { (_, eventTypes) ->
            eventType in eventTypes
        }?.key ?: throw IllegalArgumentException("No topic found for event type $eventType")
    }
}
```

6. **Pub/Sub to Cloud Run Authentication:**
   To authenticate Cloud Run with Pub/Sub without using explicit credentials, you can leverage **Identity-based Authentication** via IAM. Grant the appropriate role to your Cloud Run service:

   - In Google Cloud Console, grant the `roles/pubsub.publisher` IAM role to the **Cloud Run service account**. This allows your service to publish messages to the Pub/Sub topics.

   ```bash
   gcloud projects add-iam-policy-binding <YOUR_PROJECT_ID> \
       --member="serviceAccount:<YOUR_CLOUD_RUN_SERVICE_ACCOUNT>" \
       --role="roles/pubsub.publisher"
   ```

   Cloud Run will automatically use the default service account's credentials to authenticate with Pub/Sub when calling the publisher.

### Key Considerations for 10 TPS:
- **Reusing Publishers:** This approach reuses the Pub/Sub publishers for each topic to avoid the overhead of creating a new publisher for each request.
- **Asynchronous Publishing:** Although the `runBlocking` block is used to ensure the message is published within a 10-second timeout, publishers in Pub/Sub are asynchronous by default, which helps to meet your 10 TPS goal.
- **Auto-scaling in Cloud Run:** Ensure that Cloud Run is configured with the proper concurrency settings to handle 10 TPS. Use the Cloud Run settings to auto-scale based on load.

### Conclusion:
This setup allows you to manage new topics and CloudEvent mappings by simply updating the `application.yml` file. Authentication between Cloud Run and Pub/Sub is securely handled by IAM, eliminating the need for explicit credentials.
