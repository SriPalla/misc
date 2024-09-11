In Kotlin with Spring Boot, when you're trying to inject a `Map<String, List<String>>` from the `application.yml` file, the standard approach is to use `@ConfigurationProperties`, not `@Value`, because `@Value` doesn't handle complex types well (like maps or lists). You can resolve this issue by using the following approach:

### Steps to Properly Inject `Map<String, List<String>>` from `application.yml`

1. **Define Configuration in `application.yml`:**
   
Make sure your YAML structure is properly formatted. For example:

```yaml
cloud:
  pubsub:
    mappings:
      topicA: 
        - eventTypeA1
        - eventTypeA2
      topicB: 
        - eventTypeB1
        - eventTypeB2
```

2. **Create a Configuration Class with `@ConfigurationProperties`:**

   You should use `@ConfigurationProperties` to map the YAML properties into a Kotlin data class or a regular class.

```kotlin
import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.stereotype.Component

@Component
@ConfigurationProperties(prefix = "cloud.pubsub")
class PubSubConfig {
    lateinit var mappings: Map<String, List<String>>
}
```

In this class, the `mappings` field will be automatically populated from the YAML structure.

3. **Enable Configuration Properties in Your Application:**

   You need to make sure that the `@ConfigurationProperties` mechanism is enabled. In a Spring Boot project, you can do this by annotating your main application class with `@EnableConfigurationProperties`, although it is often enabled by default.

```kotlin
import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication

@SpringBootApplication
class Application

fun main(args: Array<String>) {
    SpringApplication.run(Application::class.java, *args)
}
```

4. **Access the Injected Configuration:**

   Now you can inject your `PubSubConfig` class into any Spring component and access the `mappings` field:

```kotlin
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController

@RestController
class CloudEventController(
    private val pubSubConfig: PubSubConfig
) {

    @GetMapping("/config")
    fun getMappings(): Map<String, List<String>> {
        return pubSubConfig.mappings
    }
}
```

This controller will return the `mappings` from the `application.yml` file when the `/config` endpoint is accessed.

### Important Notes:

- **`lateinit` vs. `var` for Maps:** We use `lateinit var` or `lateinit var mappings: Map<String, List<String>>` in Kotlin to defer the initialization of the `mappings` property until Spring injects it.
  
- **Ensure YAML is Correctly Formatted:** Make sure the YAML structure correctly reflects the types you expect. Indentation errors can cause unexpected issues.

By using `@ConfigurationProperties`, you ensure that the YAML is properly parsed and injected into the Kotlin class as expected. This approach works for more complex structures like `Map<String, List<String>>` without needing to handle conversions manually.
