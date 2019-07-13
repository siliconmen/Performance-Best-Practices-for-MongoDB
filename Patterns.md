### 应用程序模式
MongoDB是一个非常易扩展的数据，由于其动态的模式和丰富的查询模型。系统提供了广泛的二级索引功能，以优化查询性能。用户应当考虑系统的灵活性和复杂性，以便为他们的应用做出权衡。以下考虑因素将帮助你优化应用程序模式。

**发布更新，仅修改已更改的字段。** 不是在应用程序中检索整个文档，更新字段，然后将文档保存回数据库。而是向特定字段发出更新。这样做有更少的网络使用和减少数据库开销的优点。

**查询中避免否定。** 像大多数数据库系统一样，MongoDB不索引值的缺失，并且否定条件可能需要扫面所有文档。如果否定是唯一的条件，和它不是选择性的（例如，查询一个orders表，其中99%的订单已经完成，以识别那些没有完成的订单），所有的记录需要被扫描。

**尽可能使用覆盖查询。** [覆盖查询](https://docs.mongodb.com/manual/core/query-optimization/#covered-query)从索引中直接返回结果，而不用访问文档，因此非常高效。覆盖查询，查询中包含的所有字段必须出现在索引中，查询返回的所有字段也必须出现在该索引中。要判定查询是否为覆盖查询，可使用[explain()](https://docs.mongodb.com/manual/reference/method/cursor.explain/)方法。如果explain()对indexOnly字段输出显示为true，则查询由索引覆盖。MongoDB可以只查询该索引来匹配查询并返回结果。

**使用explain()测试应用程序中的每个查询。** MongoDB提供了[explain](https://docs.mongodb.com/manual/reference/method/cursor.explain/)计划功能，显示查询将如何进行或如何被解决的信息，包括：
- 返回文档的数量
- 读取文档的数量
- 使用了哪些索引
- 是否查询被覆盖，意味着没有文档需要被读取就可以返回结果
- 是否执行内存排序，表明索引是有益的
- 扫描索引条目的数量
- 查询解析所需要的时间（以毫秒为单位）（使用[executionState](https://docs.mongodb.com/manual/reference/method/cursor.explain/#executionstats-mode)模式）
- 哪些选择查询计划被拒绝（使用[allPlansExecution](https://docs.mongodb.com/manual/reference/method/cursor.explain/#allplansexecution-mode)模式）

如果查询在小于1毫秒内被解决，explian plan 将显示9毫秒。这在调优良好的系统中很常见。调用explain plan时，将放弃先前缓存的额查询计划。测试多个索引的过程被重复，以确保使用了最佳的计划。查询计划可以被计算和返回，而不必首先运行查询。这使数据库应用系统检查将使用哪个计划来执行查询，而不必等待查询运行完成。

[MongoDB Compass](https://www.mongodb.com/products/compass)提供了可视化解释计划的能力，显示一个查询执行的关键信息——例如返回的文档数量、执行时间、索引使用情况等等。执行管道的每个阶段都表示为树中的一个节点，这使得分布在多个节点的查询中查看explain计划变得更加简单。

**一个操作中更新多个数组元素。** 使用完全表达性的数组更新，开发者可以在一次更新操作中对匹配的数组元素（包括嵌套数组中的元素）执行复杂的数组操作。使用arrayFilters选项，update可以指定要修改数组字段中的哪些元素。

**避免分散查询。** 在共享系统中，查询不能被路由到单个分片，必须广播到多个分片进行评估。因为这些查询涉及到每个请求的多个分片，不能很好地扩展，会添加更多的分片。

**选择适当的写保证。** MongoDB允许管理员，向数据库发起写操作时指定持久性保证的级别，这被称作写关注[write concern](https://docs.mongodb.com/manual/core/replica-set-write-concern/)。以下选项，可以根据每个连接、每个数据库、每个集合、甚至每个操作基础进行配置。选项如下：
- WriteAcknowledged: 这是默认的写关注。mongod将确认写操作的执行，允许客户端捕获网络、复制秘钥、文档验证和其他异常。
- Journal Acknowledged: mongod在将操作刷新到主日志之后
才确认写操作。确认写操作可以在mongod崩溃后继续运行，并确保写操作在磁盘上是持久的。
- Replica Acknowledged: 它是可能的等待其他副本集成员的写入的确认。MongoDB支持写入特定数量的副本。确保写入secondaries的日志。因为副本集可以跨数据中心的框架和跨多个数据中心部署，确保写传播到其他副本集可以提供非常健壮地持久性。
- Majority: 写确认等待写应用于大多数复制集成员。确保写入记录在这些副本的日志中——包括主副本上。
- Data Center Awareness: 使用标签集，复杂的策略用来确保，在确认成功之前，将数据写到特定的副本组合中。~~例如，你可以创建一个策略，该策略要求至少向两个大路上的三个数据中心写操作，或者再一个特定数据中心的两个机架上跨服务器写操作。~~更多新查看关于[Data Center Awareness](http://docs.mongodb.com/manual/data-center-awareness/)MongoDB官文文档

**选择正确的读确认。** 为了确保隔离性和一致性，可以将[readConcern](https://docs.mongodb.com/manual/reference/read-concern/)设置为majority，指示当数据复制到副本集中的大多数节点时，应该将数据返回给应用程序，所以在发生故障时不能回滚数据。

MongoDB支持”线性化“的readConcern级别。线性化读确认确保一个节点在读取时仍然是复制集的主成员，并且如果随后选择另一个节点作为新的主成员，则不会回滚它返回的数据。配置读确认级别可能对延产生重大影响，因此应该提供[]()值，以便超时长时间运行的操作。

**需要时使用因果一致性。** MongoDB 3.6中引入的因果一致性，保证了客户端会话中每一个读操作都将始终看到前面的写操作。而不管哪个副本正在服务于请求。只在需要的地方，使用因果一致性来最小化任何延迟影响。

**使用来自MongoDB最新的驱动程序。** MongoDB支持[近12种程序语言的驱动程序](https://docs.mongodb.com/ecosystem/drivers/)。这些驱动程序由维护数据库内核的同一团队设计。驱动程序比数据更新地更频繁，通常每两个月更新一次。尽可能使用最新版本的驱动程序。如果你的程序语言可用，请安装原生扩展。加入[MongoDB community mailing list](https://groups.google.com/forum/#!forum/mongodb-dev)来跟踪更新。

**保证分片键（shard keys）的均匀分布。** 当分片键不是均匀分布用于读写时，操作可能受到单个切片容量的限制。当分片键均匀分布时，没有单个分片会限制系统的容量。

**适当的时候使用基于哈希的分片(hash-based sharding)** 对于发出基于范围查询的应用程序，基于范围的分片是有益的，因为操作被路由到最少的必要分片，通常是单个分片。然而，基于范围的分片需要对数据和查询有更好理解，某些情况下不实用。[Hash-based sharding](https://docs.mongodb.com/manual/core/index-hashed/)确保读写的统一分布，但没有提供有效的基于范围的操作