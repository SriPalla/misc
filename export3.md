To enforce properties to be defined in the `application.yml` file for a Spring Boot JAR that will be included in other projects, you can follow these steps. The goal is to make sure that the required properties are provided and raise an error if they are missing. This can be achieved using `@ConfigurationProperties` with validation annotations and integrating with Spring Boot’s property validation mechanisms.

### 1. Use `@ConfigurationProperties` with Validation

You can define a configuration properties class that will load the properties from the `application.yml` file. To enforce that specific properties are provided, you can use validation annotations such as `@NotNull`.

First, add the necessary dependencies for validation in your `build.gradle.kts`:

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter")
}
```

### 2. Create the Configuration Properties Class with Validation

Define a configuration properties class that reads and validates the required properties. Use `@NotNull` or other validation annotations to enforce the presence of required properties.

For example, create a `SftpProperties.kt` class with validation:

```kotlin
package com.example.sftp

import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.validation.annotation.Validated
import jakarta.validation.constraints.NotNull

@Validated
@ConfigurationProperties(prefix = "sftp")
data class SftpProperties(
    @field:NotNull(message = "SFTP host must be defined")
    var host: String? = null,

    @field:NotNull(message = "SFTP port must be defined")
    var port: Int? = null,

    @field:NotNull(message = "SFTP username must be defined")
    var username: String? = null,

    @field:NotNull(message = "SFTP password must be defined")
    var password: String? = null,

    var remoteDirectory: String = "/"
)
```

In this class, fields like `host`, `port`, `username`, and `password` are annotated with `@NotNull`. If any of these properties are missing in the `application.yml` file, Spring Boot will throw a validation error during startup.

### 3. Enable Configuration Properties in a Spring Configuration Class

Make sure to enable your `@ConfigurationProperties` class in the configuration. You can do this by using the `@EnableConfigurationProperties` annotation in your configuration class or Spring Boot application class:

```kotlin
package com.example.sftp

import org.springframework.boot.context.properties.EnableConfigurationProperties
import org.springframework.context.annotation.Configuration

@Configuration
@EnableConfigurationProperties(SftpProperties::class)
class SftpConfig
```

### 4. Define the Required Properties in `application.yml`

In any Spring Boot project that includes your JAR, you’ll need to define the required properties in the `application.yml` file. If the properties are not provided, Spring Boot will fail to start with an error message indicating which properties are missing.

```yaml
sftp:
  host: your-sftp-host
  port: 22
  username: your-username
  password: your-password
  remoteDirectory: /optional/remote/directory
```

### 5. Handle Missing Properties

When properties are missing or invalid, Spring Boot will fail the startup with a descriptive message. For example, if the `host` property is missing, the startup error will look like this:

```
javax.validation.ConstraintViolationException: Validation failed for classes [com.example.sftp.SftpProperties] during startup. 
Property: sftp.host
Message: SFTP host must be defined
```

This ensures that the application won’t start unless all required properties are provided.

### 6. Optionally Provide Default Values

If certain properties are optional, you can provide default values in the configuration properties class:

```kotlin
data class SftpProperties(
    @field:NotNull(message = "SFTP host must be defined")
    var host: String? = null,

    @field:NotNull(message = "SFTP port must be defined")
    var port: Int? = 22, // Default value

    @field:NotNull(message = "SFTP username must be defined")
    var username: String? = null,

    @field:NotNull(message = "SFTP password must be defined")
    var password: String? = null,

    var remoteDirectory: String = "/" // Default value
)
```

This way, if the `port` or `remoteDirectory` properties are not provided in `application.yml`, the defaults will be used.

### Summary

1. **Use `@ConfigurationProperties` with validation annotations** (`@NotNull`, etc.) to enforce required properties.
2. **Enable configuration properties** using `@EnableConfigurationProperties` in your configuration class.
3. **Define required properties** in `application.yml` when including the JAR in other applications.
4. If any required properties are missing, Spring Boot will fail to start with an appropriate validation error message.

This approach ensures that properties required for your JAR are properly enforced across all applications using it.
