# 可插拔的MongoDB存储引擎

MongoDB公开了存储引擎API，以支持可插拔存储引擎（给MongoDB带来可扩展性的新能力）的集成，并实现特定硬件架构的最佳使用，以满足特定负载要求。 MongoDB附带多个支持的存储引擎：

- 默认的**WiredTiger存储引擎**。对于大多数应用而言，WiredTiger的粒度并发控制(granular concurrency control)和本地压缩(native compression)为绝大多数的应用程序提供最佳的综合性能和存储效率。
- **加密存储引擎**，可保护高度敏感的数据，且无需为文件系统加密付出性能或管理开销。加密存储引擎基于WiredTiger，因此在本文档中，有关WiredTiger的语句也同样适用于加密存储引擎。该引擎是[MongoDB Enterprise Advanced](https://www.mongodb.com/products/mongodb-enterprise-advanced)的一部分。
- **内存存储引擎**，可为最苛刻的应用提供可预测的延迟和实时分析。该引擎是[MongoDB Enterprise Advanced](https://www.mongodb.com/products/mongodb-enterprise-advanced)的一部分。
- **MMAPv1存储引擎**，仅用于向后兼容。 MongoDB 4.0版本不推荐使用此引擎。

任何这些存储引擎都可以在任意一个MongoDB副本集中共存，从而可以轻松地在它们之间进行评测和迁移。WiredTiger是MongoDB部署的默认存储引擎。如果把其他引擎作为首选，则使用`--storageEngine`选项启动mongod。如果启动了3.2（或更高版本）的mongod进程并且一个或多个数据库已经存在，则MongoDB将使用这些数据库创建时指定存储引擎。

虽然每个存储引擎都针对不同的工作负载进行了优化，但用户仍然可以使用相同MongoDB查询语言，数据模型，扩展，安全性和操作工具(这与当前使用的引擎无关)。因此，本指南中的大多数最佳实践适用于所有(MongoDB支持的)存储引擎。不同存储引擎(使用)建议上的任何差异都将被记录。