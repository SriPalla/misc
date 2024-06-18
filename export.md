Running a MongoDB replica set for local development using Docker Compose on an Apple Silicon (M1/M2) chipset can be challenging due to compatibility issues. However, you can still achieve this by setting up the Docker Compose configuration correctly. Here's a step-by-step guide to help you set up a MongoDB replica set using Docker Compose:

### Step-by-Step Guide

#### 1. Create the Docker Compose File

Create a `docker-compose.yml` file to define the MongoDB services. We'll create three MongoDB instances to form a replica set.

```yaml
version: '3.8'
services:
  mongo1:
    image: mongo:4.4.3
    container_name: mongo1
    ports:
      - "27017:27017"
    volumes:
      - mongo1-data:/data/db
      - ./initiateReplicaSet.sh:/docker-entrypoint-initdb.d/initiateReplicaSet.sh
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=example
      - MONGO_REPLICA_SET_NAME=rs0

  mongo2:
    image: mongo:4.4.3
    container_name: mongo2
    ports:
      - "27018:27017"
    volumes:
      - mongo2-data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=example
      - MONGO_REPLICA_SET_NAME=rs0

  mongo3:
    image: mongo:4.4.3
    container_name: mongo3
    ports:
      - "27019:27017"
    volumes:
      - mongo3-data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=example
      - MONGO_REPLICA_SET_NAME=rs0

volumes:
  mongo1-data:
  mongo2-data:
  mongo3-data:
```

#### 2. Create the Initialization Script

Create a script named `initiateReplicaSet.sh` in the same directory as your `docker-compose.yml` file. This script will initiate the replica set.

```sh
#!/bin/bash
echo "Waiting for MongoDB to start..."
until mongo --host mongo1:27017 --eval "print(\"waited for connection\")"
do
    sleep 5
done

echo "Initiating replica set..."
mongo --host mongo1:27017 <<EOF
rs.initiate(
  {
    _id: "rs0",
    members: [
      { _id: 0, host: "mongo1:27017" },
      { _id: 1, host: "mongo2:27017" },
      { _id: 2, host: "mongo3:27017" }
    ]
  }
)
EOF

echo "Replica set initiated."
```

Make sure the script is executable:

```sh
chmod +x initiateReplicaSet.sh
```

#### 3. Build and Run the Docker Containers

Run the following command in the directory containing your `docker-compose.yml` and `initiateReplicaSet.sh` files:

```sh
docker-compose up -d
```

### Connecting to the Replica Set from Spring Boot

Update your `application.properties` or `application.yml` to use the MongoDB replica set:

**application.properties**:

```properties
spring.data.mongodb.uri=mongodb://root:example@mongo1:27017,mongo2:27017,mongo3:27017/yourDatabaseName?replicaSet=rs0&retryWrites=false
```

**application.yml**:

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://root:example@mongo1:27017,mongo2:27017,mongo3:27017/yourDatabaseName?replicaSet=rs0&retryWrites=false
```

Replace `yourDatabaseName` with the name of your database.

### Additional Notes

- **Compatibility**: Ensure you are using a MongoDB Docker image that is compatible with the Apple Silicon chipset. The `mongo:4.4.3` image should work, but if you encounter issues, consider trying a different version or a custom-built image for ARM architecture.
- **Network Configuration**: Ensure that your Docker setup allows for the necessary network communication.
- **Retry Mechanism**: If you still face issues, consider adding a retry mechanism or increasing the sleep duration in the script to ensure that MongoDB has enough time to start up and initiate the replica set properly.

This setup should allow you to run a MongoDB replica set locally on an Apple Silicon chipset using Docker Compose, ensuring that your Spring Data transactions can work correctly.
