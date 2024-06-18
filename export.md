Yes, you can pass a JavaScript file to the MongoDB shell (`mongosh` or `mongo`) and execute it. This is particularly useful for running scripts that contain multiple commands or more complex logic.

Here’s how you can do it:

### Using a JavaScript File with `mongosh`

1. **Create the JavaScript File**

Create a JavaScript file, for example, `initiateReplicaSet.js`, with the commands you want to run:

```js
// initiateReplicaSet.js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "localhost:27017" }
  ]
});

var primaryElected = false;
while (!primaryElected) {
  var status = rs.status();
  status.members.forEach(member => {
    if (member.stateStr === "PRIMARY") {
      primaryElected = true;
    }
  });
  sleep(1000);
}

print("Primary elected, creating customer database...");

db = db.getSiblingDB("customer");
db.createCollection("customers");
print("Customer database and collection created.");
```

2. **Run the Script Using `mongosh`**

You can pass this JavaScript file to the `mongosh` using the `--file` option:

```sh
mongosh --host <hostname>:<port> -u <username> -p <password> --authenticationDatabase <auth-db> /path/to/initiateReplicaSet.js
```

Replace `<hostname>`, `<port>`, `<username>`, `<password>`, `<auth-db>`, and `/path/to/initiateReplicaSet.js` with your actual MongoDB server details, authentication credentials, and the path to your JavaScript file.

### Example with Docker Compose

To run this script automatically when starting a Docker container, you can use a `docker-compose.yml` file. Here's how you can set it up:

1. **Directory Structure**

Assume you have the following directory structure:

```
project-root/
  |- docker-compose.yml
  |- initiateReplicaSet.js
```

2. **Docker Compose File**

Create or update your `docker-compose.yml` file to include a service that runs the MongoDB client and executes the JavaScript file:

```yaml
version: '3.8'
services:
  mongodb:
    image: mongo:5.0.5
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    volumes:
      - mongo-data:/data/db

  mongo-client:
    image: mongo:5.0.5
    container_name: mongo-client
    depends_on:
      - mongodb
    volumes:
      - ./initiateReplicaSet.js:/docker-entrypoint-initdb.d/initiateReplicaSet.js
    entrypoint: ["mongosh", "--host", "mongodb", "--port", "27017", "-u", "root", "-p", "example", "--authenticationDatabase", "admin", "/docker-entrypoint-initdb.d/initiateReplicaSet.js"]

volumes:
  mongo-data:
```

### Explanation

- **MongoDB Service (`mongodb`)**: This service runs the MongoDB server.
- **MongoDB Client Service (`mongo-client`)**:
  - Uses the official `mongo` image.
  - Mounts the `initiateReplicaSet.js` script from the host to the container.
  - Uses the `entrypoint` to run `mongosh` with the script.

### Running the Containers

To start the containers and run the initialization script, use:

```sh
docker-compose up -d
```

This command starts the MongoDB server and then runs the `mongosh` client to execute the `initiateReplicaSet.js` script.

### Final Notes

- Ensure the JavaScript file (`initiateReplicaSet.js`) has the correct commands and logic you need to initialize your MongoDB setup.
- Adjust the MongoDB server details and authentication credentials as needed.
- This approach is useful for setting up MongoDB environments automatically using Docker and Docker Compose.
