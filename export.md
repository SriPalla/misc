To stub a `WebClient` POST request using `mockk` in Kotlin, you can mock the `WebClient` and chain the method calls to simulate a POST request and return a stubbed response.

Here’s an example of how to do this:

### 1. Add dependencies for MockK and WebClient

Make sure you have the following dependencies in your `build.gradle.kts` (for Gradle with Kotlin DSL):

```kotlin
testImplementation("io.mockk:mockk:1.13.3") // Use the latest version of MockK
testImplementation("org.springframework.boot:spring-boot-starter-webflux")
testImplementation("org.springframework.boot:spring-boot-starter-test") {
    exclude(group = "org.mockito") // Exclude Mockito to avoid conflicts
}
```

### 2. Mock the WebClient POST request

Here’s an example test case where you mock the `WebClient` to stub a POST request:

```kotlin
import io.mockk.every
import io.mockk.mockk
import io.mockk.slot
import io.mockk.verify
import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Test
import org.springframework.web.reactive.function.BodyInserters
import org.springframework.web.reactive.function.client.WebClient
import org.springframework.web.reactive.function.client.WebClientResponseException
import reactor.core.publisher.Mono

class WebClientMockkTest {

    private val webClient: WebClient = mockk()

    @Test
    fun `should mock WebClient post request`() {
        // Mocking the request specification
        val requestBodySpec: WebClient.RequestBodySpec = mockk()
        val requestHeadersSpec: WebClient.RequestHeadersSpec<*> = mockk()
        val responseSpec: WebClient.ResponseSpec = mockk()

        // Mocking the WebClient chain
        every { webClient.post() } returns requestBodySpec
        every { requestBodySpec.uri(any<String>()) } returns requestBodySpec
        every { requestBodySpec.body(any<BodyInserters.BodyInserter<*, *>>()) } returns requestHeadersSpec
        every { requestHeadersSpec.retrieve() } returns responseSpec

        // Mocking the response
        val expectedResponse = "Response"
        every { responseSpec.bodyToMono(String::class.java) } returns Mono.just(expectedResponse)

        // Code under test
        val actualResponse = webClient.post()
            .uri("/test-endpoint")
            .body(BodyInserters.fromValue("Request Body"))
            .retrieve()
            .bodyToMono(String::class.java)
            .block()

        // Assertions
        assertEquals(expectedResponse, actualResponse)

        // Verify the interaction
        verify { webClient.post() }
        verify { requestBodySpec.uri("/test-endpoint") }
        verify { requestBodySpec.body(any<BodyInserters.BodyInserter<*, *>>()) }
        verify { requestHeadersSpec.retrieve() }
        verify { responseSpec.bodyToMono(String::class.java) }
    }
}
```

### Explanation:

1. **Mock WebClient components**: `mockk()` is used to mock `WebClient`, `RequestBodySpec`, `RequestHeadersSpec`, and `ResponseSpec`.
2. **Stub method calls**: Use `every { ... } returns ...` to mock method calls in the WebClient chain.
3. **Simulate response**: We return `Mono.just(expectedResponse)` from the `bodyToMono` method to simulate the response.
4. **Verify interactions**: After the test runs, use `verify { ... }` to ensure that the WebClient's methods were called with the expected parameters.

This approach allows you to fully stub and control the behavior of the `WebClient` for testing purposes using `mockk`.
