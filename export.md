To set up Azure App Configuration with Spring Boot 3.2.3 using Kotlin, including auto refresh interval and integrating with Azure Feature Management and Actuator health, you'll need to configure your `build.gradle.kts` file appropriately. Below is an example setup that demonstrates how to achieve this.

### 1. Add Dependencies

Make sure to include the required dependencies for Azure App Configuration, Azure Feature Management, Spring Boot, Actuator, and Kotlin support.

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    kotlin("jvm") version "1.5.31"
    id("org.springframework.boot") version "3.2.3"
}

group = "com.example"
version = "1.0-SNAPSHOT"
java.sourceCompatibility = JavaVersion.VERSION_11

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("com.azure.spring:azure-spring-boot-starter-appconfiguration:3.2.3")
    implementation("com.microsoft.azure:azure-actuator:1.2.0")
    implementation("com.microsoft.azure:azure-feature-management-spring-boot-starter:2.3.0")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    implementation("org.jetbrains.kotlin:kotlin-stdlib")
}

tasks.withType<KotlinCompile> {
    kotlinOptions {
        freeCompilerArgs = listOf("-Xjsr305=strict")
        jvmTarget = "11"
    }
}
```

### 2. Application Configuration

In your Spring Boot application, you'll need to configure Azure App Configuration properties in `application.properties` or `application.yml`.

```properties
# Azure App Configuration
spring.cloud.azure.appconfiguration.connection-string=Endpoint=https://your-app-config.azconfig.io;Id=your-id;Secret=your-secret
spring.cloud.azure.appconfiguration.refresh-interval=5s

# Azure Feature Management
feature-management.enabled=true
```

Replace `https://your-app-config.azconfig.io`, `your-id`, and `your-secret` with your actual Azure App Configuration endpoint and credentials.

### 3. Enable Actuator Endpoints

Ensure Actuator endpoints are enabled in your `application.properties` or `application.yml`:

```properties
management.endpoints.web.exposure.include=*
```

### 4. Kotlin Application

Create a simple Kotlin Spring Boot application.

```kotlin
import org.springframework.beans.factory.annotation.Value
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController

@SpringBootApplication
class MyApp

fun main(args: Array<String>) {
    runApplication<MyApp>(*args)
}

@RestController
class MyController {

    @Value("\${featureFlag.enabled:false}")
    private lateinit var featureEnabled: String

    @GetMapping("/hello")
    fun hello(): String {
        return "Hello World! Feature is enabled: $featureEnabled"
    }
}
```

In this example, `MyController` demonstrates retrieving a feature flag value from Azure Feature Management.

### 5. Actuator Health Endpoint

To check the health of your application including Azure App Configuration status, access the Actuator health endpoint, usually at `/actuator/health`.

That's a basic setup to integrate Azure App Configuration, Azure Feature Management, Actuator, and Kotlin with Spring Boot 3.2.3. Adjustments may be needed based on your specific requirements and environment. Replace placeholders with actual values according to your Azure setup.
