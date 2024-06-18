To define a health check for the `mongodb/mongodb-atlas-local` Docker image in a Docker Compose file, you can use the `healthcheck` instruction. The health check typically involves running a command that verifies the MongoDB server is responsive. Here's how you can set it up:

### Example Docker Compose File with Health Check

Assuming you have a directory structure like this:

```
project-root/
  |- docker-compose.yml
  |- init-mongo.js
```

### 1. Initialization Script

**init-mongo.js**:

```js
// init-mongo.js
db = db.getSiblingDB("customer");
db.createCollection("customers");
print("Customer database and collection created.");
```

### 2. Docker Compose File

Create or update your `docker-compose.yml` file to include the health check for the `mongodb/mongodb-atlas-local` image.

```yaml
version: '3.8'
services:
  mongodb:
    image: mongodb/mongodb-atlas-local:latest
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=example
    volumes:
      - mongo-data:/data/db
      - ./init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js
    healthcheck:
      test: ["CMD-SHELL", "mongosh --eval 'db.adminCommand(\"ping\")' || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5
    entrypoint: ["/bin/bash", "-c", "mongod --bind_ip_all --auth && sleep 5 && mongosh --host localhost:27017 -u root -p example --authenticationDatabase admin /docker-entrypoint-initdb.d/init-mongo.js"]

volumes:
  mongo-data:
```

### Explanation

- **MongoDB Service (`mongodb`)**:
  - Uses the `mongodb/mongodb-atlas-local` image.
  - Sets environment variables for the MongoDB root user.
  - Mounts the initialization script from the host to the container.
  - Defines a health check that runs a `ping` command to the MongoDB server every 10 seconds, with a timeout of 5 seconds and retries up to 5 times.
  - Uses `entrypoint` to start the MongoDB server, waits a few seconds for the server to initialize, and then runs the initialization script (`init-mongo.js`).

### Running the Containers

To start the containers and apply the health check, use:

```sh
docker-compose up -d
```

### Health Check Details

- **Test Command**: `mongosh --eval 'db.adminCommand("ping")' || exit 1` - This command checks if the MongoDB server is responsive by issuing a `ping` command.
- **Interval**: Runs the health check every 10 seconds.
- **Timeout**: Each health check must complete within 5 seconds.
- **Retries**: The container will be considered unhealthy after 5 consecutive failures.

### Notes

- **Initialization Script**: The script `init-mongo.js` will run after MongoDB starts, creating the `customer` database and `customers` collection.
- **Customizing Health Check**: Adjust the health check parameters (`interval`, `timeout`, `retries`) as needed for your specific use case.

This setup ensures that your MongoDB instance using the `mongodb-atlas-local` image is properly initialized and health-checked, making it suitable for local development and testing environments.
