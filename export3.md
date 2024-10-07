To create a reusable Spring Boot library for SFTP using the `JSch` library, you can build a standalone library (JAR) that provides configurable beans to manage SFTP connections and file uploads. The beans can then be imported and used in other Spring Boot applications.

### Step 1: Set Up the Gradle Project

1. Create a new Gradle project.
2. Define the necessary dependencies in `build.gradle.kts`.

```kotlin
plugins {
    kotlin("jvm") version "1.8.21"
    id("org.springframework.boot") version "3.3.3"
    id("io.spring.dependency-management") version "1.1.0"
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter")
    implementation("org.springframework.boot:spring-boot-starter-configuration-processor")
    implementation("com.jcraft:jsch:0.1.55")
}
```

### Step 2: Create SFTP Configuration Properties

Define a configuration properties class that can be used to inject SFTP settings into the Spring Boot application. Create `SftpProperties.kt`:

```kotlin
package com.example.sftp

import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.stereotype.Component

@Component
@ConfigurationProperties(prefix = "sftp")
data class SftpProperties(
    var host: String? = null,
    var port: Int = 22,
    var username: String? = null,
    var password: String? = null,
    var remoteDirectory: String = "/"
)
```

### Step 3: Create SFTP Connection Factory

Create an SFTP connection factory to manage SFTP sessions using JSch. This bean will be reusable across multiple Spring Boot applications. Create `SftpConnectionFactory.kt`:

```kotlin
package com.example.sftp

import com.jcraft.jsch.ChannelSftp
import com.jcraft.jsch.JSch
import com.jcraft.jsch.Session
import org.springframework.stereotype.Component

@Component
class SftpConnectionFactory(private val sftpProperties: SftpProperties) {

    fun createSession(): ChannelSftp {
        val jsch = JSch()
        val session = jsch.getSession(sftpProperties.username, sftpProperties.host, sftpProperties.port)
        session.setPassword(sftpProperties.password)
        session.setConfig("StrictHostKeyChecking", "no") // Disable host key checking for simplicity
        session.connect()

        val channel = session.openChannel("sftp") as ChannelSftp
        channel.connect()
        return channel
    }

    fun disconnectSession(channelSftp: ChannelSftp) {
        channelSftp.disconnect()
        channelSftp.session.disconnect()
    }
}
```

### Step 4: Create SFTP Service for File Upload

Create a service that uploads files to the SFTP server. This service will use the `SftpConnectionFactory` to manage connections. Create `SftpService.kt`:

```kotlin
package com.example.sftp

import com.jcraft.jsch.ChannelSftp
import org.springframework.stereotype.Service
import java.io.File
import java.io.InputStream

@Service
class SftpService(private val sftpConnectionFactory: SftpConnectionFactory) {

    fun uploadFile(localFile: File, remoteFileName: String, remoteDirectory: String? = null) {
        val remoteDir = remoteDirectory ?: sftpConnectionFactory.sftpProperties.remoteDirectory
        val channelSftp = sftpConnectionFactory.createSession()
        
        try {
            channelSftp.cd(remoteDir)
            channelSftp.put(localFile.absolutePath, remoteFileName)
        } finally {
            sftpConnectionFactory.disconnectSession(channelSftp)
        }
    }

    fun uploadFile(inputStream: InputStream, remoteFileName: String, remoteDirectory: String? = null) {
        val remoteDir = remoteDirectory ?: sftpConnectionFactory.sftpProperties.remoteDirectory
        val channelSftp = sftpConnectionFactory.createSession()

        try {
            channelSftp.cd(remoteDir)
            channelSftp.put(inputStream, remoteFileName)
        } finally {
            sftpConnectionFactory.disconnectSession(channelSftp)
        }
    }
}
```

### Step 5: Configure SFTP Properties in the Application

You will need to specify the SFTP credentials and other configurations in `application.yml`:

```yaml
sftp:
  host: your-sftp-host
  port: 22
  username: your-username
  password: your-password
  remoteDirectory: /optional/remote/directory
```

### Step 6: Package the Library

Package the library as a JAR by running the following Gradle command:

```bash
./gradlew build
```

The resulting JAR can now be included in any other Spring Boot application.

### Step 7: Use the Library in Another Spring Boot Application

To use the library in another Spring Boot application, you can include the JAR in your `build.gradle.kts` dependencies. Suppose you've published your JAR to a local Maven repository or a central one.

```kotlin
dependencies {
    implementation("com.example.sftp:sftp-library:1.0.0") // Replace with actual coordinates
}
```

You can then inject the `SftpService` bean and use it to upload files:

```kotlin
package com.example.app

import com.example.sftp.SftpService
import org.springframework.boot.CommandLineRunner
import org.springframework.stereotype.Component
import java.io.File

@Component
class AppRunner(private val sftpService: SftpService) : CommandLineRunner {
    override fun run(vararg args: String?) {
        val localFile = File("path/to/local/file.txt") // Replace with your local file path
        sftpService.uploadFile(localFile, "uploaded_file.txt")
    }
}
```

### Summary

This approach creates a reusable SFTP service using `JSch` that can be packaged as a Spring Boot library (JAR). The library includes an SFTP connection factory and a service for uploading files. The credentials and default remote directory are configurable via `application.yml`. Once packaged, the library can be imported and used in any other Spring Boot application.
