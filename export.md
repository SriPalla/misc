Azure Cosmos DB supports multiple APIs for different data models, including the SQL API (for NoSQL), MongoDB API, Cassandra API, Gremlin API, and Table API. Each API is designed to provide compatibility with different database technologies. Here, we'll focus on the differences between Cosmos DB for NoSQL (using the SQL API) and Cosmos DB for MongoDB (using the MongoDB API).

### Cosmos DB for NoSQL (SQL API)

#### Features:
1. **Query Language**:
   - Uses a SQL-like query language specifically designed for Cosmos DB.
   - Supports rich querying capabilities including JOINs, aggregations, and user-defined functions.

2. **Data Model**:
   - Document-based, storing JSON documents.
   - Supports flexible schema and hierarchical data structures.

3. **Indexing**:
   - Automatic indexing of all properties by default, with options for manual indexing policies.
   - Supports custom indexing policies for better performance optimization.

4. **Consistency Levels**:
   - Provides five consistency levels: Strong, Bounded Staleness, Session, Consistent Prefix, and Eventual.

5. **Integration and SDKs**:
   - Natively supported by Azure SDKs for various programming languages (Java, .NET, Python, Node.js, etc.).
   - Full support for Azure features like Azure Functions, Logic Apps, and Data Factory.

6. **Unique Key Constraints**:
   - Allows unique key constraints to enforce uniqueness on one or more fields within a logical partition.

#### Use Cases:
- Applications requiring advanced querying capabilities and integration with other Azure services.
- Scenarios where flexible schema and hierarchical data models are beneficial.
- Workloads that benefit from rich indexing and custom indexing policies.

### Cosmos DB for MongoDB (MongoDB API)

#### Features:
1. **Query Language**:
   - Uses MongoDB query language, providing compatibility with MongoDB clients and tools.
   - Supports MongoDB CRUD operations and aggregation framework.

2. **Data Model**:
   - Document-based, storing BSON (Binary JSON) documents.
   - Compatible with existing MongoDB applications with minimal changes.

3. **Indexing**:
   - Supports MongoDB indexing capabilities, including compound indexes, text indexes, and geospatial indexes.
   - Automatic indexing of the `_id` field and manual creation of additional indexes as needed.

4. **Consistency Levels**:
   - Supports MongoDB's default consistency model (eventual consistency).
   - Can be configured to use Cosmos DB's consistency levels for specific operations.

5. **Integration and SDKs**:
   - Compatible with MongoDB drivers and tools (e.g., MongoDB shell, MongoDB Compass, Mongoose).
   - Enables existing MongoDB applications to migrate to Cosmos DB with minimal code changes.

6. **Unique Key Constraints**:
   - Supports unique indexes to enforce uniqueness on one or more fields.

#### Use Cases:
- Existing MongoDB applications looking to migrate to a globally distributed database with minimal changes.
- Applications requiring compatibility with MongoDB tools and drivers.
- Scenarios where MongoDB's query language and data model are preferred.

### Key Differences

1. **API and Query Language**:
   - **NoSQL (SQL API)**: Uses a SQL-like query language specific to Cosmos DB.
   - **MongoDB API**: Uses MongoDB's query language, providing compatibility with MongoDB tools and drivers.

2. **Indexing**:
   - **NoSQL (SQL API)**: Automatic indexing of all properties, with custom indexing policies.
   - **MongoDB API**: Supports MongoDB indexing strategies, with manual index creation.

3. **Consistency Levels**:
   - **NoSQL (SQL API)**: Offers five consistency levels.
   - **MongoDB API**: Default MongoDB consistency with optional Cosmos DB consistency levels.

4. **Integration**:
   - **NoSQL (SQL API)**: Deep integration with Azure services and SDKs.
   - **MongoDB API**: Compatibility with MongoDB ecosystem, including drivers and tools.

5. **Unique Key Constraints**:
   - Both APIs support unique key constraints, but they are defined and managed differently.

### Choosing Between Cosmos DB for NoSQL and MongoDB

- **Use Cosmos DB for NoSQL (SQL API) if**:
  - You need advanced querying capabilities with a SQL-like language.
  - Your application benefits from tight integration with Azure services.
  - You require flexible consistency levels and rich indexing options.

- **Use Cosmos DB for MongoDB (MongoDB API) if**:
  - You are migrating an existing MongoDB application and want to minimize code changes.
  - You prefer MongoDB's query language and data model.
  - Your application relies on MongoDB tools and drivers.

### Conclusion

Both Cosmos DB for NoSQL and Cosmos DB for MongoDB offer robust, globally distributed database solutions with unique features tailored to different needs. Your choice should depend on your application's requirements, existing technology stack, and specific use cases.
