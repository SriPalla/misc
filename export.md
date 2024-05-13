To configure Azure App Configuration with Spring Boot 3.2.3 and Kotlin, including automatic configuration refresh and integration with Actuator health, you'll need to use the `azure-appconfiguration-actuator` and `azure-spring-cloud-starter-appconfiguration-config` dependencies. Below is an example `build.gradle.kts` file that demonstrates how to set up these configurations:

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    id("org.springframework.boot") version "3.2.3"
    kotlin("jvm") version "1.6.10"
}

group = "com.example"
version = "1.0-SNAPSHOT"
java.sourceCompatibility = JavaVersion.VERSION_11

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("com.microsoft.azure:azure-spring-cloud-starter-appconfiguration-config:1.4.1")
    implementation("com.microsoft.azure:azure-appconfiguration-actuator:1.4.1")
    implementation("org.jetbrains.kotlin:kotlin-stdlib")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.0")

    implementation("org.springframework.boot:spring-boot-starter-actuator")
}

tasks.withType<KotlinCompile> {
    kotlinOptions {
        freeCompilerArgs = listOf("-Xjsr305=strict")
        jvmTarget = "11"
    }
}

bootJar {
    archiveFileName = "your-application.jar"
}

// Azure App Configuration properties
azureAppConfiguration {
    connectionString = "your-connection-string"
}

// Spring Boot application properties
springBoot {
    buildInfo()
}

// Actuator endpoints configuration
management {
    endpoints {
        web {
            exposure {
                include = listOf("*")
            }
        }
    }
}
```

In this `build.gradle.kts` file:

- We're using the Spring Boot Gradle plugin (`id("org.springframework.boot") version "3.2.3"`) to enable Spring Boot features.
- The `kotlin("jvm") version "1.6.10"` plugin is added to support Kotlin compilation.
- Dependencies include `spring-boot-starter-web` for web-based applications, `azure-spring-cloud-starter-appconfiguration-config` and `azure-appconfiguration-actuator` for Azure App Configuration integration and Actuator endpoints.
- Kotlin standard library (`kotlin-stdlib`) and reflection library (`kotlin-reflect`) are included.
- `kotlinx-coroutines-core` is added for coroutines support.
- `spring-boot-starter-actuator` is included for Spring Boot Actuator endpoints.
- `azureAppConfiguration` block specifies the Azure App Configuration connection string.
- `springBoot` block is used to configure Spring Boot properties.
- `management` block configures Actuator endpoints exposure.

Replace `"your-connection-string"` in `azureAppConfiguration` with your Azure App Configuration connection string. This `build.gradle.kts` file sets up a basic Spring Boot application with Azure App Configuration integration and Actuator endpoints exposed. Make sure to replace `"your-application.jar"` in `bootJar` with your desired application name.

After configuring your `build.gradle.kts` file, you can start integrating Azure App Configuration properties into your Spring Boot application using the `@Value` annotation or the Azure App Configuration SDK for Java/Kotlin. Additionally, Actuator health and other endpoints (`/actuator/health`) will be available by default for monitoring and management purposes.
