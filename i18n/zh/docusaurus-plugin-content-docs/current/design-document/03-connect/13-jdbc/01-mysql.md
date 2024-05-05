# MySQL JDBC Connector

## MySQL JDBC Source Connector: 从MySQL到EventMesh 

1. 启动MySQL和Apache EventMesh Runtime。
2. 检查配置**`source-config.yml`** 以及在 配置文件 **`server-config.yml`** 中启用jdbc source connector
3. 启动 **`JdbcConnectorServer`** 服务，将MySQL指定表中的数据发送到EventMesh的对应Topic

```yaml
pubSubConfig:
  meshAddress: 127.0.0.1:10001
  subject: TopicTest
  idc: FT
  env: PRD
  group: rocketmqSource
  appId: 5032
  userName: jdbcSourceUser
  passWord: jdbcPassWord
sourceConnectorConfig:
  maxTask: 10 #max task number
  batchMaxRows: 100
  skipSnapshot: false
  snapshotMaxThreads: 10
  snapshotSchema: true
  snapshotData: true
  snapshotFetchSize: 100
  databaseType: mysql
  databaseIncludeList:
  databaseExcludeList:
  tableIncludeList:
  tableExcludeList:
  jdbcConfig:
    hostname: localhost
    port: 3306
    user: root
    password: xxxx
    initialStatements:
    connectTimeout: 10
  mysqlConfig:
    serverId: 123
    keepAlive: true
    keepAliveInterval: 6000
offsetStorageConfig:
  offsetStorageType: nacos
  offsetStorageAddr: 127.0.0.1:8848
  extensions: {
    #same with topic
    dataId: TopicTest,
    #same with group
    group: rocketmqSource
  }

```

## MySQL设置

在使用Apache EventMesh MySQL JDBC Connector 之前需要对MySQL进行一些设置。

### 创建用户

Apache EventMesh JDBC connector需要MySQL的账户。这个MySQL的账号必须对连接器捕获数据库数据全部变动有适当的权限。

- 创建用户

  ```bash
  mysql> CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
  ```

- 设置用户权限

  ```bash
  mysql> GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'user' IDENTIFIED BY 'password';
  ```

- 刷新用户权限

  ```bash
  mysql> FLUSH PRIVILEGES;
  ```

> 说明： 如果是测试可以使用root账号，省略创建用户的过程

### 启用MySQL的Binlog

MySQL JDBC Source Connector是基于MySQL复制binlog日志为基础实现，因此需要启用MySQL的binlog日志。

- 检查log-bin是否开启

  ```bash
  mysql>SHOW VARIABLES LIKE 'log_bin';
  
  //或者MySql 5.x
  mysql>SELECT variable_value as "Binlog Status" FROM information_schema.global_variables WHERE variable_name='log_bin';
  
  //或者 MySql 8.x
  mysql> SELECT variable_value as "Binlog Status" FROM performance_schema.global_variables WHERE variable_name='log_bin';
  ```

- 如果查询出来的值是**`OFF`** 请更新MySQL的配置文件开启binlog日志文件

  ```properties
  server-id         = 223344 #Mysql服务ID
  log_bin                     = mysql-bin  #binlog的名称
  binlog_format               = ROW  
  binlog_row_image            = FULL
  binlog_expire_logs_seconds  = 864000
  ```

  > 配置可以参照[MySQL官网的配置](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html)

  配置说明：

  | 属性                       | 说明                                                         |
  | -------------------------- | ------------------------------------------------------------ |
  | server-id                  | mysql server id 每一个服务要求必须唯一                       |
  | log_bin                    | 设置binlog的前缀文件名称                                     |
  | binlog_format              | 必须设置为ROW                                                |
  | binlog_row_image           | 必须设置为FULL                                               |
  | binlog_expire_logs_seconds | 设置二进制日志的过期时间（秒）。在过期时间结束后，二进制日志文件可以自动删除。可能的删除发生在启动和刷新二进制日志时。 |

  

- 校验是否已经开启binlog

  ```
  mysql>SHOW VARIABLES LIKE 'log_bin';
  
  //或者MySql 5.x
  mysql>SELECT variable_value as "Binlog Status" FROM information_schema.global_variables WHERE variable_name='log_bin';
  
  //或者 MySql 8.x
  mysql> SELECT variable_value as "Binlog Status" FROM performance_schema.global_variables WHERE variable_name='log_bin';
  ```

  

### 配置MySQL GTID

//TODO,功能开发中

### 配置Session超时

在数据库快照处理的时，读取表的时候已经建立的连接有可能会超时，可以通过设置`interactive_timeout` 和 `wait_timeout` 来防止超时的发生

```bash
mysql> interactive_timeout=<duration-in-seconds>
mysql> wait_timeout=<duration-in-seconds>
```

