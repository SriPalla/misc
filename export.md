Implementing a system where a microservice deployed in a Kubernetes cluster pushes database change events to Google Pub/Sub after performing save or update operations, and running a scheduled cron job to fetch the last updated records from an Azure Cosmos DB MongoDB API collection, involves several components. Here's a detailed overview including the implementation in Kotlin Spring Boot, along with pros, cons, costs, risks, performance considerations, and dependencies.

### Implementation Steps in Kotlin Spring Boot

#### 1. Set Up Azure Cosmos DB for MongoDB API

- **Create a Cosmos DB Instance**: Ensure you have an Azure Cosmos DB instance with the MongoDB API enabled.

#### 2. Set Up Google Pub/Sub

- **Create a Pub/Sub Topic**: In Google Cloud, create a Pub/Sub topic where you will publish the change events.

#### 3. Set Up the Microservice

- **Add Dependencies**: Include the necessary dependencies in your `build.gradle` or `pom.xml` file for Spring Boot, MongoDB, Google Pub/Sub, and scheduling.

  ```groovy
  // Example for Gradle
  dependencies {
      implementation 'org.springframework.boot:spring-boot-starter'
      implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
      implementation 'com.google.cloud:google-cloud-pubsub:1.113.5'
      implementation 'org.springframework.boot:spring-boot-starter-scheduled'
  }
  ```

- **Configure MongoDB Client**: Set up the MongoDB client to connect to your Cosmos DB instance.

  ```kotlin
  import com.mongodb.client.MongoClients
  import com.mongodb.client.MongoClient
  import com.mongodb.client.MongoDatabase
  import com.mongodb.client.MongoCollection
  import org.bson.Document
  import org.springframework.stereotype.Component

  @Component
  class MongoDbClient {
      private val connectionString = "mongodb://your-connection-string"
      private val client: MongoClient = MongoClients.create(connectionString)
      private val database: MongoDatabase = client.getDatabase("your-database")
      val collection: MongoCollection<Document> = database.getCollection("your-collection")
  }
  ```

- **Configure Google Pub/Sub Client**:

  ```kotlin
  import com.google.cloud.pubsub.v1.Publisher
  import com.google.pubsub.v1.ProjectTopicName
  import com.google.pubsub.v1.PubsubMessage
  import org.springframework.beans.factory.annotation.Value
  import org.springframework.stereotype.Service

  @Service
  class PubSubService(
      @Value("\${gcp.project-id}") private val projectId: String,
      @Value("\${gcp.pubsub.topic-id}") private val topicId: String
  ) {
      private val topicName = ProjectTopicName.of(projectId, topicId)

      fun publishMessage(message: String) {
          val publisher = Publisher.newBuilder(topicName).build()
          val pubsubMessage = PubsubMessage.newBuilder().setData(ByteString.copyFromUtf8(message)).build()
          publisher.publish(pubsubMessage).addListener(
              Runnable { println("Message published: $message") },
              MoreExecutors.directExecutor()
          )
      }
  }
  ```

- **Implement Scheduled Task to Fetch and Push Changes**:

  ```kotlin
  import org.springframework.scheduling.annotation.Scheduled
  import org.springframework.stereotype.Service

  @Service
  class ChangeEventProcessorService(
      private val mongoDbClient: MongoDbClient,
      private val pubSubService: PubSubService
  ) {
      @Scheduled(cron = "0 */5 * * * *") // Every 5 minutes
      fun processChanges() {
          val now = System.currentTimeMillis()
          val oneHourAgo = now - 3600000 // 1 hour ago
          val query = Document("lastUpdated", Document("\$gte", oneHourAgo))
          val changes = mongoDbClient.collection.find(query).iterator()

          while (changes.hasNext()) {
              val change = changes.next()
              // Convert the change to a message
              val message = convertChangeToMessage(change)
              // Publish the message to Google Pub/Sub
              pubSubService.publishMessage(message)
          }
      }

      private fun convertChangeToMessage(change: Document): String {
          // Implement conversion logic
          return change.toJson()
      }
  }
  ```

#### 4. Deploy to Kubernetes

- **Create a Docker Image**: Dockerize your Spring Boot application.
- **Deploy to Kubernetes**: Use Kubernetes manifests or Helm charts to deploy your microservice to a Kubernetes cluster.

### Pros and Cons

#### Pros:
1. **Decoupling**: Decouples data processing from event publishing, allowing for scalable and maintainable architecture.
2. **Scalability**: Both the microservice and Pub/Sub system can scale independently.
3. **Flexibility**: Allows for custom logic to be applied before publishing messages to Pub/Sub.

#### Cons:
1. **Complexity**: Involves managing multiple components and ensuring their integration.
2. **Latency**: There might be delays between data changes and their processing due to the scheduled tasks.
3. **Costs**: Additional costs associated with Google Pub/Sub and potentially increased resource usage in Kubernetes.

### Costs

- **Google Pub/Sub**: Costs based on message volume, storage, and number of operations.
- **Azure Cosmos DB for MongoDB API**: Costs based on provisioned throughput (RU/s), storage, and regional replication.
- **Kubernetes**: Costs associated with running and scaling your microservice within the cluster.

### Performance

- **Change Fetching**: Performance depends on the frequency of polling and volume of changes.
- **Pub/Sub Publishing**: Google Pub/Sub is highly scalable and performant, but ensure that you handle message publishing and retries appropriately.
- **Scheduled Tasks**: The performance of scheduled tasks depends on the interval and efficiency of the change processing logic.

### Risks

- **Overlapping Executions**: Multiple instances of the scheduled task might process the same changes if not handled properly.
- **Data Consistency**: Ensure that your change processing logic is idempotent to avoid duplication or loss of data.
- **Costs**: Monitor costs associated with Pub/Sub and Cosmos DB to avoid unexpected charges.

### Dependencies

- **Google Pub/Sub**: Requires integration with Google Cloud services.
- **Azure Cosmos DB**: Requires connection to Azure Cosmos DB with MongoDB API.
- **Kubernetes**: Requires managing deployments and configurations in a Kubernetes cluster.

### Summary

Using a microservice deployed in Kubernetes to push database change events from an Azure Cosmos DB for MongoDB API collection to Google Pub/Sub involves scheduling periodic tasks to fetch updated records and publish them. This approach provides flexibility and scalability but comes with complexities and costs associated with managing multiple systems and ensuring data consistency. Implementing the solution in Kotlin Spring Boot involves setting up MongoDB and Pub/Sub clients, writing the scheduled task logic, and deploying the service to Kubernetes.
