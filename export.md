You're correct. As of now, Azure Cosmos DB for MongoDB API does not support push-based event processing through Azure Functions due to the lack of native change feed support in the MongoDB API implementation.

### Updated Comparison

### Azure Cosmos DB for NoSQL (SQL API)

#### Pros:
1. **Native API**: Optimized for performance and integrated deeply with Azure services.
2. **Flexible Schema**: Allows for a flexible schema, making it easy to store and manage diverse data types.
3. **SQL Query Language**: Familiar SQL-like query language which is easy to learn and use.
4. **Global Distribution**: Easy to replicate data across multiple regions with low latency.
5. **Multi-Model Support**: Supports key-value, document, and graph data models.
6. **Comprehensive SLAs**: Offers SLAs for throughput, latency, availability, and consistency.
7. **Integration**: Seamless integration with other Azure services like Azure Functions, Azure Logic Apps, etc.
8. **Change Feed**: Native support for change feed, enabling real-time event processing with Azure Functions.

#### Cons:
1. **Complex Pricing**: The pricing model can be complex, based on provisioned throughput (RU/s), storage, and regional replication.
2. **Learning Curve**: Despite using SQL, there are differences and additional concepts (like partitioning and RU/s) which can be a learning curve.
3. **Costs**: Can become expensive with high throughput and storage requirements.

#### Costs:
- **Provisioned Throughput**: Charged based on reserved RU/s.
- **Storage**: Charged based on the amount of data stored.
- **Other Charges**: Additional charges for multi-region writes, backups, and more.

#### Performance:
- **High Performance**: Designed for high performance with low latency and high throughput.
- **Scalability**: Automatically scales with the provisioned throughput and can handle large amounts of data with global distribution.

#### Risks:
- **Cost Overruns**: Mismanagement of provisioned throughput can lead to higher-than-expected costs.
- **Complexity**: Complexity in managing partition keys and scaling throughput.

#### Dependencies:
- **Azure Integration**: Strong dependency on the Azure ecosystem for optimal use.
- **SDKs**: Requires using Azure Cosmos DB SDKs for best performance and features.

### Azure Cosmos DB for MongoDB API

#### Pros:
1. **MongoDB Compatibility**: Supports MongoDB wire protocol, enabling applications built for MongoDB to run with little to no changes.
2. **Familiar Interface**: Familiar MongoDB API and query language, making it easy for MongoDB developers to transition.
3. **Flexible Schema**: Like MongoDB, allows for a flexible, dynamic schema.
4. **Global Distribution**: Offers the same global distribution and multi-region capabilities as other Cosmos DB APIs.
5. **Integration**: Integrates well with other Azure services.

#### Cons:
1. **Limited Features**: Some MongoDB features and commands are not supported or behave differently.
2. **Compatibility Issues**: Potential issues with specific MongoDB features or versions.
3. **Complex Pricing**: Similar to other Cosmos DB APIs, the pricing model can be complex.
4. **Costs**: Can become expensive with high throughput and storage requirements.
5. **No Change Feed**: Lacks native change feed support, making it unsuitable for scenarios requiring real-time event processing via Azure Functions.

#### Costs:
- **Provisioned Throughput**: Charged based on reserved RU/s.
- **Storage**: Charged based on the amount of data stored.
- **Other Charges**: Additional charges for multi-region writes, backups, and more.

#### Performance:
- **High Performance**: Generally high performance, but might have some performance overhead due to MongoDB compatibility layer.
- **Scalability**: Scales with provisioned throughput and can handle large data volumes with global distribution.

#### Risks:
- **Feature Parity**: Not all MongoDB features are supported, which might lead to issues if relying on unsupported features.
- **Cost Overruns**: Mismanagement of provisioned throughput can lead to higher-than-expected costs.

#### Dependencies:
- **Azure Integration**: Strong dependency on the Azure ecosystem for optimal use.
- **SDKs**: While MongoDB drivers work, using Cosmos DB SDKs can provide additional benefits.

### Event Processing

#### Push-Based Event Processing with Azure Functions

**Azure Cosmos DB for NoSQL**:
1. **Enable Change Feed**: Ensure change feed is enabled for your Cosmos DB container.
2. **Azure Function App**: Create an Azure Function App.
3. **Cosmos DB Trigger**: Add a new Azure Function with a Cosmos DB trigger using the `CosmosDBTrigger` attribute.
4. **Function Code**: Implement logic to process changes within the Azure Function.
5. **Deployment**: Deploy the function app and monitor for changes.

**Azure Cosmos DB for MongoDB**:
- **Not Supported**: Push-based event processing using change feed is not supported.

#### Pros of Using Change Feed with Azure Functions:
1. **Real-Time Processing**: Enables real-time processing of data changes.
2. **Scalable**: Functions scale automatically to handle varying loads.
3. **Integration**: Tight integration with Cosmos DB and other Azure services.

#### Cons:
1. **Latency**: There can be a slight delay between data change and function execution.
2. **Complexity**: Managing function app deployment, scaling, and monitoring can add complexity.
3. **Costs**: Additional costs for running Azure Functions, especially with high-frequency changes.

### Summary

**Azure Cosmos DB for NoSQL** is highly optimized for performance within Azure's ecosystem, offering flexible schema and strong integration with other Azure services. It supports real-time data change processing using change feed and Azure Functions.

**Azure Cosmos DB for MongoDB API** offers the advantage of MongoDB compatibility, making it easier for existing MongoDB applications to transition. However, it lacks support for push-based event processing through Azure Functions, which may limit its use in scenarios requiring real-time data processing.
