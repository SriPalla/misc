To create a Spring Boot Kotlin application as a Google Cloud Function that listens to Google Cloud Storage events and run it locally, you can follow these steps. There isn't an official local emulator for Google Cloud Functions like the `azureFunctionsRun` task, but we can simulate the function locally using `spring-cloud-function` and Spring Boot.

Here’s a complete example:

### 1. Setup the Spring Boot Kotlin Project

Add the necessary dependencies in your `build.gradle.kts`:

```kotlin
plugins {
    id("org.springframework.boot") version "3.1.0"
    id("io.spring.dependency-management") version "1.0.15.RELEASE"
    kotlin("jvm") version "1.8.20"
    kotlin("plugin.spring") version "1.8.20"
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.cloud:spring-cloud-function-adapter-gcp:3.2.8") // Google Cloud Function Adapter
    implementation("com.google.cloud.functions:functions-framework-api:1.0.4") // Google Cloud Functions API
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

### 2. Create the Storage Event Listener

Create a function that will listen to Google Cloud Storage events. Here, we use the `GoogleCloudStorageEvent`.

```kotlin
import com.google.cloud.functions.CloudEventsFunction
import com.google.cloud.storage.Storage
import io.cloudevents.CloudEvent
import org.springframework.stereotype.Component

@Component
class GcsEventListener : CloudEventsFunction {

    override fun accept(event: CloudEvent) {
        // Parse the storage event
        val storageEvent = event.data?.toBytes()?.let { String(it) } ?: "No Data"
        
        // Log event details
        println("Received Cloud Storage event: $storageEvent")
        
        // Add your business logic here
    }
}
```

### 3. Configure the Function in `application.yml`

```yaml
spring:
  cloud:
    function:
      definition: gcsEventListener
```

### 4. Running the Application Locally

Although Google Cloud Functions doesn't have a local run plugin like `azureFunctionsRun`, you can simulate the behavior locally using Spring Boot. Here's how you can run your function locally:

- **Create a Local Controller to Simulate Event Trigger**:

```kotlin
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RestController

@RestController
class LocalStorageController(val gcsEventListener: GcsEventListener) {

    @PostMapping("/simulate-gcs-event")
    fun simulateEvent(@RequestBody body: String) {
        // Simulate the event processing
        val cloudEvent = CloudEventBuilder.v1()
            .withId("1234")
            .withSource(URI.create("/local-simulation"))
            .withType("google.cloud.storage.object.finalize")
            .withDataContentType("application/json")
            .withData(body.toByteArray())
            .build()
        
        gcsEventListener.accept(cloudEvent)
    }
}
```

Now you can run your application using `./gradlew bootRun` and send a POST request to `http://localhost:8080/simulate-gcs-event` with a JSON payload to simulate the Google Cloud Storage event.

### 5. Deploy to Google Cloud Functions

Once you’re satisfied with the local testing, you can deploy the function to Google Cloud using `gcloud` CLI:

1. Package the application as a fat JAR:

    ```bash
    ./gradlew clean build
    ```

2. Deploy the function:

    ```bash
    gcloud functions deploy gcsEventListener \
        --entry-point=com.example.GcsEventListener \
        --runtime=java17 \
        --trigger-resource=<your-bucket> \
        --trigger-event=google.storage.object.finalize \
        --memory=512MB \
        --region=<your-region>
    ```

This setup allows you to develop and test the function locally and deploy it to Google Cloud Functions when ready.
