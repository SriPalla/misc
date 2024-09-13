To handle `IllegalArgumentException` in a Spring Boot Kotlin application with a `ControllerAdvice` that logs warnings and returns success while capturing details from the request body (which is a `CloudEvent`), here’s a complete example. This will show how to implement exception handling, log the request details, and ensure the flow continues with a success response.

### 1. Add Dependencies
Make sure you have the required dependencies in your `build.gradle.kts`:

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    implementation("org.springframework.boot:spring-boot-starter-logging")
    implementation("io.cloudevents:cloudevents-spring:2.3.0") // CloudEvent support
}
```

### 2. Create a CloudEvent Controller
The controller will receive a `POST` request with a `CloudEvent` in the body.

```kotlin
import io.cloudevents.CloudEvent
import org.springframework.http.ResponseEntity
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RestController

@RestController
class CloudEventController {

    @PostMapping("/cloudevent")
    fun handleCloudEvent(@RequestBody cloudEvent: CloudEvent): ResponseEntity<String> {
        if (cloudEvent.id.isNullOrEmpty()) {
            throw IllegalArgumentException("CloudEvent ID must not be null or empty")
        }
        // Process CloudEvent
        return ResponseEntity.ok("CloudEvent processed successfully")
    }
}
```

### 3. Define a ControllerAdvice for Exception Handling
The `ControllerAdvice` will handle the `IllegalArgumentException`, log the details from the `CloudEvent`, and return a success response.

```kotlin
import io.cloudevents.CloudEvent
import org.slf4j.LoggerFactory
import org.springframework.http.HttpStatus
import org.springframework.http.ResponseEntity
import org.springframework.web.bind.annotation.ControllerAdvice
import org.springframework.web.bind.annotation.ExceptionHandler
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RestController

@ControllerAdvice
@RestController
class GlobalExceptionHandler {

    private val logger = LoggerFactory.getLogger(GlobalExceptionHandler::class.java)

    @ExceptionHandler(IllegalArgumentException::class)
    fun handleIllegalArgumentException(
        ex: IllegalArgumentException, 
        @RequestBody cloudEvent: CloudEvent?
    ): ResponseEntity<String> {
        // Log a warning with details
        logger.warn("IllegalArgumentException: ${ex.message}. CloudEvent details: id=${cloudEvent?.id}, type=${cloudEvent?.type}, source=${cloudEvent?.source}")

        // Return success response despite the error
        return ResponseEntity.status(HttpStatus.OK).body("Warning: ${ex.message}, but proceeding with success")
    }
}
```

### Explanation
- **`CloudEventController`**: This defines a POST endpoint that accepts `CloudEvent` in the request body. If the `id` field is missing or empty, it throws an `IllegalArgumentException`.
- **`GlobalExceptionHandler`**: The `ControllerAdvice` catches the `IllegalArgumentException`, logs a warning with the request's `CloudEvent` details, and returns a success response (`200 OK`) with a message indicating the warning.
- **`Logger`**: The `SLF4J` logger logs the warning at the `WARN` level.

### 4. Application Configuration
Ensure your application is configured to handle exceptions and log appropriately. You can also configure logging levels in `application.yml`:

```yaml
logging:
  level:
    root: INFO
    com.example: WARN
```

### Testing with `curl`
You can test the API with the following `curl` command:

```bash
curl -X POST http://localhost:8080/cloudevent \
  -H "Content-Type: application/json" \
  -d '{
        "id": "",
        "type": "com.example.event",
        "source": "/my/source"
      }'
```

This will trigger the `IllegalArgumentException` and return a response with a warning, but still allow the flow to continue as successful.

Let me know if you need further details!
