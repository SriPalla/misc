Here's an implementation of a Spring Boot Kotlin Cloud Run app that exposes an API endpoint, accepts CloudEvent as input, and publishes it to the appropriate Pub/Sub topic based on the message type. It uses `suspend` functions to optimize for 10 TPS, with a publisher created for each topic and reused.

### 1. **application.yml**:
Define the mapping of CloudEvent types to Pub/Sub topics in `application.yml`.

```yaml
cloudEventToTopic:
  - eventType: "com.example.event.type1"
    topic: "projects/project-id/topics/topic1"
  - eventType: "com.example.event.type2"
    topic: "projects/project-id/topics/topic2"
  # Add more event-to-topic mappings as needed
```

### 2. **PubSubPublisherService**:
This service handles the creation and reuse of publishers for each topic. 

```kotlin
package com.example.cloudrun

import com.google.cloud.pubsub.v1.Publisher
import com.google.pubsub.v1.PubsubMessage
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import org.springframework.stereotype.Service
import java.util.concurrent.ConcurrentHashMap

@Service
class PubSubPublisherService {

    private val publisherMap = ConcurrentHashMap<String, Publisher>()

    suspend fun publishMessage(topic: String, message: String) {
        val publisher = getOrCreatePublisher(topic)
        val pubsubMessage = PubsubMessage.newBuilder().setData(message.toByteArray().toByteString()).build()

        withContext(Dispatchers.IO) {
            publisher.publish(pubsubMessage).get()
        }
    }

    private fun getOrCreatePublisher(topic: String): Publisher {
        return publisherMap.computeIfAbsent(topic) {
            Publisher.newBuilder(it).build()
        }
    }
}
```

### 3. **CloudEventController**:
This controller handles incoming CloudEvent requests and publishes the event to the appropriate topic.

```kotlin
package com.example.cloudrun

import io.cloudevents.CloudEvent
import io.cloudevents.spring.webflux.CloudEventHttpMessageReader
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import org.springframework.beans.factory.annotation.Value
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RestController
import reactor.core.publisher.Mono

@RestController
class CloudEventController(
    private val pubSubPublisherService: PubSubPublisherService,
    @Value("\${cloudEventToTopic}") private val eventToTopicMapping: List<EventToTopicMapping>
) {

    @PostMapping("/publish", consumes = ["application/cloudevents+json"])
    suspend fun publishEvent(@RequestBody cloudEvent: CloudEvent) {
        val eventType = cloudEvent.type
        val topic = eventToTopicMapping.find { it.eventType == eventType }?.topic
            ?: throw IllegalArgumentException("No topic mapped for event type: $eventType")

        val message = String(cloudEvent.data!!.toBytes())
        
        // Publish to the topic
        pubSubPublisherService.publishMessage(topic, message)
    }
}

data class EventToTopicMapping(
    val eventType: String,
    val topic: String
)
```

### 4. **build.gradle.kts**:
Include necessary dependencies for CloudEvents, Pub/Sub, and coroutine support.

```kotlin
plugins {
    id("org.springframework.boot") version "3.1.0"
    id("io.spring.dependency-management") version "1.1.0"
    kotlin("jvm") version "1.8.22"
    kotlin("plugin.spring") version "1.8.22"
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-webflux")
    implementation("io.cloudevents:cloudevents-spring:2.5.0")
    implementation("com.google.cloud:google-cloud-pubsub:1.118.0")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.2")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor:1.7.2")
}
```

### Explanation:
- **Controller**: The `/publish` endpoint takes in a CloudEvent, matches the event type with the topic from `application.yml`, and calls `publishMessage()` to publish the event.
- **PubSubPublisherService**: A `ConcurrentHashMap` is used to cache and reuse publishers for different topics. Publishers are created on-demand if they don’t exist for a specific topic.
- **Coroutines**: The code uses `suspend` functions and `withContext(Dispatchers.IO)` to handle I/O-bound operations like Pub/Sub publishing efficiently.
  
This setup should be optimized for handling around 10 TPS by efficiently managing publishers and leveraging Kotlin coroutines.
