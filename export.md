If the `mongodb/mongodb-atlas-local` Docker image does not include the `mongo` or `mongosh` shell, you can create a custom Docker image that adds the MongoDB shell. Here’s how you can build such an image and use it in your Docker Compose setup.

### Creating a Custom Docker Image

1. **Dockerfile**

Create a `Dockerfile` that extends the `mongodb/mongodb-atlas-local` image and adds the MongoDB shell:

```Dockerfile
# Dockerfile
FROM mongodb/mongodb-atlas-local:latest

# Install the MongoDB shell
RUN apt-get update && apt-get install -y mongo-tools

# Clean up to reduce image size
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Copy initialization scripts
COPY init-mongo.js /docker-entrypoint-initdb.d/init-mongo.js
COPY initiateReplicaSet.js /docker-entrypoint-initdb.d/initiateReplicaSet.js

# Set the entrypoint
ENTRYPOINT ["/bin/bash", "-c", "mongod --replSet rs0 --bind_ip_all --auth && sleep 5 && mongo --host localhost:27017 -u root -p example --authenticationDatabase admin /docker-entrypoint-initdb.d/initiateReplicaSet.js && mongo --host localhost:27017 -u root -p example --authenticationDatabase admin /docker-entrypoint-initdb.d/init-mongo.js"]
```

2. **Initialization Scripts**

Ensure you have your initialization scripts ready:

**initiateReplicaSet.js**:

```js
// initiateReplicaSet.js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "localhost:27017" }
  ]
});
print("Replica set initiated.");
```

**init-mongo.js**:

```js
// init-mongo.js
db = db.getSiblingDB("customer");
db.createCollection("customers");
print("Customer database and collection created.");
```

3. **Build the Custom Image**

Build the Docker image from the `Dockerfile`:

```sh
docker build -t custom-mongodb-atlas-local .
```

### Docker Compose File

Update your `docker-compose.yml` file to use the custom image and include a health check:

```yaml
version: '3.8'
services:
  mongodb:
    image: custom-mongodb-atlas-local
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=example
    volumes:
      - mongo-data:/data/db
    healthcheck:
      test: ["CMD-SHELL", "mongo --eval 'db.adminCommand(\"ping\")' || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mongo-data:
```

### Running the Containers

To start the containers and apply the health check, use:

```sh
docker-compose up -d
```

### Explanation

- **Custom Docker Image**:
  - Extends `mongodb/mongodb-atlas-local` and installs the `mongo` tools.
  - Copies the initialization scripts into the container.
  - Sets up the entrypoint to start MongoDB, initialize the replica set, and run the initialization script.

- **Health Check**:
  - Uses the `mongo` shell to ping the MongoDB server every 10 seconds, with a timeout of 5 seconds and retries up to 5 times.

This setup ensures that you have a working MongoDB instance with the necessary client tools for initialization and health checks, making it suitable for local development and testing environments.
