# 数据结构设计与索引

MongoDB使用基于BSON的二进制文档数据模型，该模型基于JSON标准。 与关系数据库中的平面表不同，MongoDB的文档数据模型与现代编程语言中使用的对象紧密对齐，并且在大多数情况下，由于为实体拥有相关数据的优点，它消除了对多文档事务或联接的需求 或包含在单个文档中的对象，而不是分布在多个表中。 有将数据建模为文档的最佳实践，正确的方法将取决于应用程序的目标。 以下注意事项将帮助您在设计应用程序的架构和索引时做出正确的选择:

- **避免文档对象数据太大**。 MongoDB中文档的最大容量为16MB。实际上，大多数文档为几千字节或更小。考虑文档比表本身更像表中的行。与其维护单个文档中的记录列表，不如使每个记录成为文档。对于大型媒体项目，例如视频或图像，请考虑使用GridFS，这是由所有驱动程序实施的约定，该约定可自动在许多较小的文档中存储二进制数据。

- **避免不必要的长字段名称**。字段名称在文档之间重复，并占用空间。通过使用较小的字段名称，您的数据将占用较少的空间，从而使大量文档可以放入RAM。请注意，使用WiredTiger的本机压缩，长字段名称对使用的磁盘空间量的影响较小，但对RAM的影响相同。

- **在考虑低基数字段的索引时要谨慎**。对低基数字段的查询可以返回大结果集。尽可能避免返回大结果集。复合索引可能包含基数较低的值，但组合字段的值应显示基数较高。

- **消除不必要的索引**。索引占用大量资源：即使启用了压缩功能，它们也会消耗RAM，并且在更新字段时必须维护其关联的索引，这会产生额外的磁盘I/O开销。

- **删除作为其他索引的前缀的索引**。复合索引可用于查询索引中的前导字段。例如，姓氏，名字的复合索引也可用于过滤查询仅指定姓氏。在此示例中，仅在姓氏上附加索引是不必要的，
