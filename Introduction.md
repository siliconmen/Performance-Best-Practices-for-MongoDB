MongoDB 设计初衷是为了满足现代应用程序的需求。其技术架构有如下特点：
    - 文档数据模型 - 将现实世界映射到数据世界的最好方式
    - 分布式系统设计 - 允许你将数据合理分布存储，避免数据丢失
    - 平台无关的统一使用体验 - 即使你切换平台，云厂商，都不会影响工作，让你的工作真正做到面向未来

本指南的主要内容是向你介绍如何在MongoDB系统下进行规模化性能提升。这里有一些关键知识点，包括硬件，应用模式，还有在 MongoDB 4.0刚刚发布的 ACID事务，模型设计与索引，硬盘I/O，Amazon EC2和性能测试设计原则等。在此期间，你可以参考 [MongoDB document](https://docs.mongodb.com/manual/) 和官方线上免费学习网站 [MongoDB University](https://university.mongodb.com/courses/catalog)。同时 MongoDB 也提供了许多咨询服务，在您使用MongoDB的过程中遇到任何问题都可以进行咨询。

本指南的目标用户是自行管理MongoDB数据库的用户。 有另一本专用指南 [MongoDB Atlas Best Practices](https://www.mongodb.com/collateral/mongodb-atlas-best-practices)是提供给直接使用云厂商MongoDB服务的用户。

关于MongoDB的体系架构及其一些基本假设的讨论，可以看 [MongoDB Architecture Guide](https://www.mongodb.com/collateral/mongodb-architecture-guide)。关于如何操作MongoDB数据库的讨论，提供了 [MongoDB Operations Best Practices](https://www.mongodb.com/collateral/mongodb-operations-best-practices)文档。