To create a reusable Spring Boot JAR that connects to an SFTP server to place a file by reading credentials in Kotlin with Gradle, you can follow these steps:

### 1. **Add Required Dependencies**
Add the necessary dependencies for SFTP (such as `spring-integration-sftp`) to your `build.gradle.kts`:

```kotlin
plugins {
    id("org.springframework.boot") version "3.1.0"
    id("io.spring.dependency-management") version "1.1.0"
    kotlin("jvm") version "1.8.0"
    kotlin("plugin.spring") version "1.8.0"
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-integration")
    implementation("org.springframework.integration:spring-integration-sftp")
    implementation("com.jcraft:jsch:0.1.55")
    implementation("org.springframework.boot:spring-boot-starter")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
}

repositories {
    mavenCentral()
}
```

### 2. **Create Configuration Class**
Create an `SftpConfig` class that sets up the SFTP session factory and gateway.

```kotlin
package com.example.sftp

import com.jcraft.jsch.ChannelSftp
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.integration.annotation.Gateway
import org.springframework.integration.annotation.MessagingGateway
import org.springframework.integration.dsl.IntegrationFlow
import org.springframework.integration.dsl.IntegrationFlows
import org.springframework.integration.file.FileHeaders
import org.springframework.integration.file.remote.gateway.AbstractRemoteFileOutboundGateway
import org.springframework.integration.file.support.FileExistsMode
import org.springframework.integration.sftp.dsl.Sftp
import org.springframework.integration.sftp.session.DefaultSftpSessionFactory
import org.springframework.messaging.Message
import java.io.File

@Configuration
class SftpConfig {

    @Bean
    fun sftpSessionFactory(): DefaultSftpSessionFactory {
        val factory = DefaultSftpSessionFactory()
        factory.setHost("sftp-host") // Replace with your host
        factory.setPort(22)
        factory.setUser("sftp-user") // Replace with your username
        factory.setPassword("sftp-password") // Replace with your password
        factory.setAllowUnknownKeys(true)
        return factory
    }

    @Bean
    fun sftpOutboundFlow(): IntegrationFlow {
        return IntegrationFlows
            .from("toSftpChannel")
            .handle(
                Sftp.outboundGateway(sftpSessionFactory(), AbstractRemoteFileOutboundGateway.Command.PUT, "payload")
                    .remoteDirectory("remote/dir/path")
                    .fileExistsMode(FileExistsMode.REPLACE)
                    .chmod(775)
            )
            .get()
    }
}
```

### 3. **Create SFTP Gateway Interface**
Use a gateway to send the file to SFTP.

```kotlin
package com.example.sftp

import org.springframework.integration.annotation.MessagingGateway
import org.springframework.stereotype.Component
import java.io.File

@MessagingGateway(defaultRequestChannel = "toSftpChannel")
interface SftpGateway {
    fun sendToSftp(file: File)
}
```

### 4. **Service for Uploading Files**
Create a service that uses the `SftpGateway` to upload a file.

```kotlin
package com.example.sftp

import org.springframework.stereotype.Service
import java.io.File

@Service
class SftpService(private val sftpGateway: SftpGateway) {

    fun uploadFile(filePath: String) {
        val file = File(filePath)
        if (file.exists()) {
            sftpGateway.sendToSftp(file)
        } else {
            throw IllegalArgumentException("File not found: $filePath")
        }
    }
}
```

### 5. **Main Application**
In the `main` function, you can test the functionality by calling the `SftpService`.

```kotlin
package com.example

import com.example.sftp.SftpService
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.context.ApplicationContext

@SpringBootApplication
class SftpApplication

fun main(args: Array<String>) {
    val context: ApplicationContext = runApplication<SftpApplication>(*args)
    val sftpService = context.getBean(SftpService::class.java)

    // Example file upload
    sftpService.uploadFile("/path/to/local/file.txt")
}
```

### 6. **Externalize Configuration**
For production use, you may want to externalize your SFTP credentials in `application.yml`:

```yaml
sftp:
  host: sftp-host
  port: 22
  username: sftp-user
  password: sftp-password
  remote-directory: /remote/dir/path
```

In your `SftpConfig`, you can use `@ConfigurationProperties` or `@Value` to inject these values from the `application.yml`.

---

This setup allows you to package the Spring Boot application as a JAR that can be reused in any project by calling the `SftpService`. You can also externalize SFTP credentials and paths via `application.yml` for flexibility.
