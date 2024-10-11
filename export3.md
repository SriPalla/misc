To create a **Spring Cloud Function for Google Cloud Platform** (GCP) that processes **Google Cloud Storage (GCS) file publish events** using Spring Boot with Kotlin, here’s a step-by-step guide:

### 1. Project Setup

First, configure your project by adding the necessary dependencies for Spring Cloud Function, Google Cloud Functions, and Google Cloud Storage in your `build.gradle.kts`.

```kotlin
plugins {
    id("org.springframework.boot") version "3.1.0"
    id("io.spring.dependency-management") version "1.0.15.RELEASE"
    kotlin("jvm") version "1.8.20"
    kotlin("plugin.spring") version "1.8.20"
}

group = "com.example"
version = "1.0.0-SNAPSHOT"
java.sourceCompatibility = JavaVersion.VERSION_17

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter")
    implementation("org.springframework.cloud:spring-cloud-function-adapter-gcp:3.2.8") // GCP function adapter
    implementation("com.google.cloud.functions:functions-framework-api:1.0.4") // Google Functions API
    implementation("com.google.cloud:google-cloud-storage:2.11.0") // GCS client
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.withType<KotlinCompile> {
    kotlinOptions {
        freeCompilerArgs = listOf("-Xjsr305=strict")
        jvmTarget = "17"
    }
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

### 2. Create the GCS Event Listener Function

Define a **function** that listens to new file events (GCS Object Finalize events) and processes them.

Create a `StorageEventListener.kt` class:

```kotlin
package com.example.demo

import com.google.cloud.functions.CloudEventsFunction
import com.google.cloud.storage.Storage
import io.cloudevents.CloudEvent
import org.springframework.stereotype.Component
import com.google.cloud.storage.StorageOptions
import org.slf4j.LoggerFactory

@Component
class StorageEventListener : CloudEventsFunction {

    private val logger = LoggerFactory.getLogger(StorageEventListener::class.java)
    private val storage: Storage = StorageOptions.getDefaultInstance().service

    override fun accept(event: CloudEvent) {
        val bucket = event.attributes["bucket"] ?: "unknown"
        val file = event.attributes["name"] ?: "unknown"

        logger.info("New file published to bucket $bucket: $file")

        // Fetch file metadata or content from GCS
        val blob = storage.get(bucket, file)

        if (blob != null) {
            // Example: Read the file content
            val content = String(blob.getContent())
            logger.info("File content: $content")
        } else {
            logger.warn("File not found: $file")
        }
    }
}
```

This class processes the new file event by retrieving the file name and bucket from the event's `CloudEvent` attributes. It fetches the file content from GCS using the `Google Cloud Storage` client and logs the file content (or processes it as needed).

### 3. `application.yml` Configuration

In your `application.yml`, you can configure the function name if needed:

```yaml
spring:
  cloud:
    function:
      definition: storageEventListener
```

### 4. Running the Application Locally

Since you can't fully simulate GCP Storage events locally, you can create a simple REST controller to simulate the event trigger:

Create a `LocalEventSimulator.kt` class:

```kotlin
package com.example.demo

import io.cloudevents.CloudEvent
import io.cloudevents.core.builder.CloudEventBuilder
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RestController
import java.net.URI

@RestController
class LocalEventSimulator(val storageEventListener: StorageEventListener) {

    @PostMapping("/simulate-event")
    fun simulateEvent(@RequestBody eventPayload: Map<String, String>) {
        val bucket = eventPayload["bucket"] ?: "test-bucket"
        val name = eventPayload["name"] ?: "test-file.txt"

        // Simulating a CloudEvent for a new file in GCS
        val cloudEvent: CloudEvent = CloudEventBuilder.v1()
            .withId("12345")
            .withSource(URI.create("https://example.com/storage"))
            .withType("google.storage.object.finalize")
            .withDataContentType("application/json")
            .withAttribute("bucket", bucket)
            .withAttribute("name", name)
            .build()

        storageEventListener.accept(cloudEvent)
    }
}
```

You can simulate the GCS event locally by posting to the `/simulate-event` endpoint with a sample payload, such as:

```json
{
  "bucket": "my-bucket",
  "name": "my-file.txt"
}
```

Run the application locally using:

```bash
./gradlew bootRun
```

### 5. Deploying the Function to Google Cloud Functions

Once the function works locally, you can deploy it to Google Cloud Functions.

First, build the project:

```bash
./gradlew clean build
```

Then deploy it to Google Cloud Functions:

```bash
gcloud functions deploy processStorageEvent \
    --entry-point=com.example.demo.StorageEventListener \
    --runtime=java17 \
    --trigger-event=google.storage.object.finalize \
    --trigger-resource=<YOUR_BUCKET_NAME> \
    --memory=512MB \
    --region=<YOUR_REGION>
```

- Replace `<YOUR_BUCKET_NAME>` with the actual GCS bucket you want to listen to.
- Replace `<YOUR_REGION>` with your preferred GCP region.

### 6. Testing the Google Cloud Function

To test, upload a file to the specified GCS bucket. The function will automatically trigger and process the new file event.

You can also check logs for function execution via:

```bash
gcloud functions logs read processStorageEvent
```

### Conclusion

This example demonstrates how to create a **Spring Boot Kotlin** application using **Spring Cloud Function** that listens to **Google Cloud Storage** events, processes newly uploaded files, and runs as a **Google Cloud Function**.
