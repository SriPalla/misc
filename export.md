Running a MongoDB replica set on Docker Desktop for Apple Silicon (M1 or M2) requires configuring your Docker Compose setup properly. Here’s a step-by-step guide to ensure your MongoDB replica set works smoothly for local development:

### Step-by-Step Guide

#### 1. Create the Docker Compose File

Create a `docker-compose.yml` file to define the MongoDB services. This example will set up a three-node replica set.

```yaml
version: '3.8'
services:
  mongo1:
    image: mongo:5.0.5
    container_name: mongo1
    ports:
      - "27017:27017"
    volumes:
      - mongo1-data:/data/db
      - ./initiateReplicaSet.sh:/docker-entrypoint-initdb.d/initiateReplicaSet.sh
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=example
    command: ["mongod", "--replSet", "rs0"]

  mongo2:
    image: mongo:5.0.5
    container_name: mongo2
    ports:
      - "27018:27017"
    volumes:
      - mongo2-data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=example
    command: ["mongod", "--replSet", "rs0"]

  mongo3:
    image: mongo:5.0.5
    container_name: mongo3
    ports:
      - "27019:27017"
    volumes:
      - mongo3-data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=example
    command: ["mongod", "--replSet", "rs0"]

volumes:
  mongo1-data:
  mongo2-data:
  mongo3-data:
```

#### 2. Create the Initialization Script

Create a script named `initiateReplicaSet.sh` in the same directory as your `docker-compose.yml` file. This script will initiate the replica set.

```sh
#!/bin/bash
# Wait for MongoDB to start
echo "Waiting for MongoDB to start..."
until mongo --host mongo1:27017 --eval "print(\"waited for connection\")"
do
    sleep 5
done

# Initiate the replica set
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

# Wait until the replica set is initiated and a primary is elected
until mongo --host mongo1:27017 --eval "rs.status().members[0].stateStr" | grep PRIMARY
do
    sleep 5
done

echo "Replica set initiated and primary elected."
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

#### 4. Verify the Replica Set

After starting the containers, verify the replica set status. You can connect to the MongoDB shell of `mongo1` and check the replica set status:

```sh
docker exec -it mongo1 mongo --eval "rs.status()"
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

- **Compatibility**: The `mongo` image should be compatible with Apple Silicon. If you encounter any issues, try using a more recent version of MongoDB that supports ARM architecture.
- **Network Configuration**: Ensure that your Docker setup allows for the necessary network communication between containers.
- **Initialization Timing**: The script includes a waiting mechanism to confirm that a primary has been elected, which should prevent the "timeout while waiting for a server that matches" error.
- **Ports**: Adjust the ports if necessary to avoid conflicts with other services running on your machine.

This setup should provide a MongoDB replica set running in Docker on your Apple Silicon MacBook, ready for local development with Spring Data transactions.
