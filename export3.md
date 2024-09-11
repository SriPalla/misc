If `@ConfigurationProperties` is still not working after following the troubleshooting steps, you can use a more manual approach to construct a `Map` from configuration properties using `Environment` and Kotlin’s utility functions. This approach provides more control and can be useful when dealing with issues related to property binding.

### Manual Approach Using `Environment`

You can manually fetch properties from the `Environment` and construct the `Map` as needed. This method involves accessing properties directly and parsing them into the required format.

#### Example Setup

1. **Define Properties in `application.yml`**

   Ensure your properties are defined in a way that you can easily parse them.

   **Example `application.yml`:**

   ```yaml
   topics:
     topic1: eventTypeA,eventTypeB
     topic2: eventTypeC
   ```

2. **Create a Service to Parse Properties**

   You can create a service that retrieves and parses these properties from the `Environment` bean.

   **Example Kotlin Service:**

   ```kotlin
   import org.springframework.core.env.Environment
   import org.springframework.stereotype.Service

   @Service
   class TopicsService(private val environment: Environment) {

       fun getTopics(): Map<String, List<String>> {
           val topicsMap = mutableMapOf<String, List<String>>()

           // Retrieve properties as a list of keys
           val topicKeys = environment.propertySources
               .flatMap { ps -> ps.source.keys }
               .filter { it.startsWith("topics.") }
               .map { it as String }

           for (key in topicKeys) {
               val topic = key.substringAfter("topics.")
               val events = environment.getProperty(key, String::class.java)
                   ?.split(",")
                   ?.map { it.trim() }
                   ?: emptyList()

               topicsMap[topic] = events
           }

           return topicsMap
       }
   }
   ```

3. **Inject and Use the Service**

   Use the `TopicsService` in your controllers or other components to access the parsed configuration.

   **Example Controller:**

   ```kotlin
   import org.springframework.web.bind.annotation.GetMapping
   import org.springframework.web.bind.annotation.RequestMapping
   import org.springframework.web.bind.annotation.RestController

   @RestController
   @RequestMapping("/topics")
   class TopicsController(private val topicsService: TopicsService) {

       @GetMapping
       fun getTopics(): Map<String, List<String>> {
           return topicsService.getTopics()
       }
   }
   ```

### Explanation

- **Environment Access:** The `Environment` bean provides access to properties defined in your configuration files. By directly querying it, you can handle properties in a more controlled manner.
- **Property Parsing:** The `TopicsService` manually retrieves properties, splits them, and constructs a `Map` based on your configuration.
- **Flexible Parsing:** This approach allows you to handle properties flexibly and can be adjusted according to the format of your configuration.

### Alternative: Use `@Value` with SpEL

If you prefer to keep things simple, you can use Spring Expression Language (SpEL) to directly parse and convert properties in your service or component.

**Example Service Using `@Value`:**

```kotlin
import org.springframework.beans.factory.annotation.Value
import org.springframework.stereotype.Service

@Service
class TopicsService {

    @Value("\${topics.topic1}")
    private lateinit var topic1: String

    @Value("\${topics.topic2}")
    private lateinit var topic2: String

    fun getTopics(): Map<String, List<String>> {
        return mapOf(
            "topic1" to topic1.split(",").map { it.trim() },
            "topic2" to topic2.split(",").map { it.trim() }
        )
    }
}
```

### Conclusion

If `@ConfigurationProperties` is not working, manually parsing properties from the `Environment` or using `@Value` with SpEL can be effective alternatives. These methods provide more control and flexibility for constructing configuration data into the required format.
