To package your Spring Boot application as a ZIP file specifically for Google Cloud Functions using a Gradle plugin, you can use the `com.google.cloud.functions` Gradle plugin. This plugin simplifies the process of creating deployable artifacts for Google Cloud Functions.

### Step-by-Step Guide

Here's how to set it up in your project:

### 1. Set Up Your Gradle Build File

1. **Add the Google Cloud Functions Plugin**: First, add the `google-cloud-functions-gradle-plugin` to your `build.gradle.kts` file.

```kotlin
plugins {
    id("org.jetbrains.kotlin.jvm") version "1.8.20"
    id("org.springframework.boot") version "3.1.0"
    id("io.spring.dependency-management") version "1.0.15.RELEASE"
    id("com.google.cloud.functions") version "3.0.0" // Add this line
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
    implementation("com.google.cloud:google-cloud-storage:2.11.0")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

// Specify the main class for the application
application {
    mainClass.set("com.example.demo.ApplicationKt") // Replace with your main class
}
```

### 2. Create a Function Entry Point

Ensure that your main function or entry point class is properly defined. It should look something like this:

```kotlin
package com.example.demo

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class Application

fun main(args: Array<String>) {
    runApplication<Application>(*args)
}
```

### 3. Configure the Functions Plugin

Configure the `functions` plugin to define how to package your function:

```kotlin
googleCloudFunction {
    functionName = "processStorageEvent" // The name of your cloud function
    entryPoint = "com.example.demo.StorageEventListener" // The entry point for the function
    runtime = "java17" // Runtime version
    memory = 512 // Memory allocation
    triggerHttp = false // Set to true if you want an HTTP trigger
    triggerResource = "<YOUR_BUCKET_NAME>" // For storage event triggers, set your bucket name
    triggerEvent = "google.storage.object.finalize" // The event that triggers the function
}
```

### 4. Create the ZIP Artifact

You can create a ZIP file with the following command:

```bash
./gradlew clean build functions:package
```

This command compiles your code, creates the JAR file, and then packages it as a ZIP artifact for Google Cloud Functions.

### 5. Deploy to Google Cloud Functions

After packaging your function, you can deploy it using the `gcloud` command:

```bash
gcloud functions deploy processStorageEvent \
    --entry-point=com.example.demo.StorageEventListener \
    --runtime=java17 \
    --trigger-event=google.storage.object.finalize \
    --trigger-resource=<YOUR_BUCKET_NAME> \
    --source=build/distributions/your-function.zip \
    --memory=512MB \
    --region=<YOUR_REGION>
```

- Replace `<YOUR_BUCKET_NAME>` with your Google Cloud Storage bucket name.
- Replace `<YOUR_REGION>` with your desired GCP region.

### Example of Complete `build.gradle.kts`

Here’s how your complete `build.gradle.kts` might look:

```kotlin
plugins {
    id("org.jetbrains.kotlin.jvm") version "1.8.20"
    id("org.springframework.boot") version "3.1.0"
    id("io.spring.dependency-management") version "1.0.15.RELEASE"
    id("com.google.cloud.functions") version "3.0.0" // Google Cloud Functions plugin
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
    implementation("com.google.cloud:google-cloud-storage:2.11.0")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

googleCloudFunction {
    functionName = "processStorageEvent"
    entryPoint = "com.example.demo.StorageEventListener"
    runtime = "java17"
    memory = 512
    triggerHttp = false
    triggerResource = "<YOUR_BUCKET_NAME>" // Set your bucket name
    triggerEvent = "google.storage.object.finalize" // Storage event
}
```

### Conclusion

Using the Google Cloud Functions Gradle plugin, you can easily package your Spring Boot application as a ZIP artifact and deploy it to Google Cloud Functions. Make sure to update the `triggerResource` and `entryPoint` properties to match your application’s specific requirements.
