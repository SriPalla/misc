The Gradle equivalent of the `com.google.cloud.functions.functions-maven-plugin` for packaging and deploying Google Cloud Functions is primarily achieved through the **Functions Framework**. While there's no direct one-to-one equivalent of the Maven plugin, you can achieve similar functionality using Gradle tasks.

### Setting Up Google Cloud Functions with Gradle

Here's how to set up your Gradle project to package a Google Cloud Function using the Functions Framework.

#### 1. Add Dependencies

In your `build.gradle.kts`, include the Functions Framework dependency and other required dependencies:

```kotlin
plugins {
    id("org.jetbrains.kotlin.jvm") version "1.8.20"
    id("org.springframework.boot") version "3.1.0"
    id("io.spring.dependency-management") version "1.0.15.RELEASE"
}

group = "com.example"
version = "1.0.0"
java.sourceCompatibility = JavaVersion.VERSION_17

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter")
    implementation("org.springframework.cloud:spring-cloud-function-adapter-gcp:3.2.8")
    implementation("com.google.cloud.functions:functions-framework-api:1.0.0") // Functions Framework API
    implementation("com.google.cloud:google-cloud-storage:2.11.0")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}
```

#### 2. Create the Function Entry Point

Ensure that your function has an entry point, typically in a class like this:

```kotlin
package com.example.demo

import com.google.cloud.functions.HttpFunction
import com.google.cloud.functions.HttpRequest
import com.google.cloud.functions.HttpResponse
import java.io.PrintWriter

class StorageEventListener : HttpFunction {
    override fun service(request: HttpRequest, response: HttpResponse) {
        response.contentType = "text/plain"
        PrintWriter(response.writer).use { writer ->
            writer.println("Hello, this is a Cloud Function responding to an event!")
        }
    }
}
```

#### 3. Create a Custom Gradle Task for Packaging

You can create a custom Gradle task to package your application as a ZIP file for deployment:

```kotlin
tasks.register<Zip>("zipFunction") {
    from(tasks.jar.get().archiveFile) // Include the built JAR
    archiveFileName.set("function.zip")
    destinationDirectory.set(file("$buildDir/libs"))
}
```

#### 4. Build the Project and Create the ZIP File

Run the following command to build your project and create the ZIP artifact:

```bash
./gradlew clean build zipFunction
```

This command generates a ZIP file in the `build/libs/` directory.

#### 5. Deploy to Google Cloud Functions

To deploy your ZIP file to Google Cloud Functions, use the following command:

```bash
gcloud functions deploy processStorageEvent \
    --entry-point=com.example.demo.StorageEventListener \
    --runtime=java17 \
    --trigger-http \
    --source=build/libs/function.zip \
    --memory=512MB \
    --region=<YOUR_REGION>
```

### Conclusion

While there isn't a direct equivalent of the `functions-maven-plugin` in Gradle, you can use the Functions Framework along with Gradle tasks to package and deploy your Google Cloud Functions effectively. This approach allows you to customize your build process while leveraging the Functions Framework capabilities. If you have further questions or need assistance, feel free to ask!
