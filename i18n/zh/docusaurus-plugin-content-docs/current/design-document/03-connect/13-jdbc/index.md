# Apache EventMesh JDBC 连接器

## 概述

Apache EventMesh JDBC 连接器可以使用多个源事件，然后通过JDBC驱动程序写入关系数据库，Apache EventMesh JDBC连接器支持多种数据方言，包括：MySQL, Oracle(WIP), PostgreSQL(WIP) 等

## JDBC连接器的工作原理

![jdbc-connector架构](C:\Users\ljbmx\Documents\jdbc-connector架构.png)

JDBC Source连接器通过处理源数据源的二进制数据转换成EventMesh内的数据格式发送到Apache EventMesh Runtime中。然后JDBC Sink连接器获取到Apache EventMesh Runtime的数据，将数据转换成目标数据库的数据格式通过JDBC将其写入目标数据库。

![实现逻辑](C:\Users\ljbmx\Documents\实现逻辑.png)

JDBC Source 连接器通过数据转换器将数据库的CDC数据转换成EventMesh JDBC引擎能够处理的数据，通过Antlr4 将数据解析为JDBC 的数据。然后通过转换器将CDC的数据转换成对应的EventMesh JDBC对应的事件。最终将数据发送到Apache EventMesh Runtime 设置的对应主题中。

JDBC Sink连接器订阅Apache EventMesh Runtime Source发送的数据的主题，将事件转换成对应的JDBC数据。然后根据不同的事件将JDBC数据插入目标数据中。



