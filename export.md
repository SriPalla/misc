To create a reusable JAR in Kotlin for a Spring Boot project that connects to an SFTP server to upload files, you can follow these steps. The credentials will be read from application properties, and if no remote directory is passed during the upload, it will default to `/`.

### Step-by-Step Guide:

#### 1. Add Dependencies to `build.gradle.kts`
You'll need the necessary dependencies for the SFTP connection and Spring Boot.

```kotlin
plugins {
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.0"
    kotlin("jvm") version "1.8.21"
    kotlin("plugin.spring") version "1.8.21"
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter")
    implementation("org.springframework.boot:spring-boot-starter-integration")
    implementation("org.springframework.integration:spring-integration-sftp")
    implementation("com.jcraft:jsch:0.1.55") // SFTP library
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}
```

#### 2. Create Configuration Class for SFTP
This class will configure the SFTP session factory and gateway to handle file uploads.

```kotlin
import com.jcraft.jsch.ChannelSftp
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.integration.annotation.IntegrationComponentScan
import org.springframework.integration.file.remote.session.SessionFactory
import org.springframework.integration.sftp.session.DefaultSftpSessionFactory

@Configuration
@IntegrationComponentScan
class SftpConfig {

    @Bean
    fun sftpSessionFactory(): SessionFactory<ChannelSftp.LsEntry> {
        val factory = DefaultSftpSessionFactory(true)
        factory.setHost("your.sftp.host") // or read from application.yml
        factory.setPort(22)
        factory.setUser("your-username")
        factory.setPassword("your-password")
        factory.setAllowUnknownKeys(true)
        return factory
    }
}
```

#### 3. Create the SFTP Gateway Interface

```kotlin
import org.springframework.integration.annotation.MessagingGateway
import org.springframework.integration.file.remote.gateway.AbstractRemoteFileOutboundGateway
import org.springframework.messaging.handler.annotation.Payload
import java.io.File

@MessagingGateway
interface SftpGateway {

    @AbstractRemoteFileOutboundGateway.Command("put")
    fun uploadToSftp(@Payload file: File, remoteDir: String = "/")
}
```

#### 4. Create the Service to Handle File Upload
This service will be responsible for uploading files to the SFTP server.

```kotlin
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.stereotype.Service
import java.io.File

@Service
class SftpService @Autowired constructor(private val sftpGateway: SftpGateway) {

    fun uploadFile(file: File, remoteDir: String? = null) {
        val directory = remoteDir ?: "/"
        sftpGateway.uploadToSftp(file, directory)
    }
}
```

#### 5. Application Properties
You can store the credentials and other configuration properties in the `application.yml` file.

```yaml
sftp:
  host: your.sftp.host
  port: 22
  username: your-username
  password: your-password
```

#### 6. Using the Service

In your Spring Boot application or any service:

```kotlin
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.CommandLineRunner
import org.springframework.stereotype.Component
import java.io.File

@Component
class FileUploader @Autowired constructor(private val sftpService: SftpService) : CommandLineRunner {
    
    override fun run(vararg args: String?) {
        val file = File("path/to/your/file.txt")
        sftpService.uploadFile(file) // Uses default "/" directory
    }
}
```

### Notes:
- By default, the directory is set to `/`. If no remote directory is passed, this default will be used.
- The `SftpConfig` class can be extended to read host, port, username, and password from `application.yml` instead of hardcoding.

### Building the JAR

Run the following command to build the JAR:

```bash
./gradlew build
```

This will generate a JAR file that you can reuse in other projects for connecting to SFTP and uploading files.
