

> 参考文章
>
> * https://gitee.com/itCjb/spring-cloud-alibaba-seata-demo
> * https://thinkingcao.blog.csdn.net/article/details/109320221



## 安装seata

> 在安装了nacos的前提下

### 第一步

下载seata-1.3.0

https://codeload.github.com/seata/seata/zip/v1.3.0

下载seata-server-1.4.1

https://phoenixnap.dl.sourceforge.net/project/seata.mirror/v1.4.1/seata-server-1.4.1.zip

### 第二步

在你的参与全局事务的数据库中加入undo_log这张表

```mysql
CREATE TABLE IF NOT EXISTS `undo_log`
(
    `branch_id`     BIGINT(20)   NOT NULL COMMENT 'branch transaction id',
    `xid`           VARCHAR(100) NOT NULL COMMENT 'global transaction id',
    `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
    `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
    `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
    `log_created`   DATETIME(6)  NOT NULL COMMENT 'create datetime',
    `log_modified`  DATETIME(6)  NOT NULL COMMENT 'modify datetime',
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8 COMMENT ='AT transaction mode undo table';
```



在你的mysql数据库中创建名为seata的库,并使用以下下sql

```mysql
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
```

```mysql
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
```

```mysql
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(96),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
```

### 第三部

在你的项目中引入seata依赖

如果你的微服务是dubbo:

```xml
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
```

如果你是springcloud

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <version>2.2.2.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>io.seata</groupId>
            <artifactId>seata-spring-boot-starter</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
```

### 第四步

从官方github仓库拿到参考配置做修改:https://github.com/seata/seata/tree/develop/script/client

加到你项目的application.yml中.

```yaml
seata:
  enabled: true
  application-id: applicationName
  tx-service-group: my_test_tx_group
  enable-auto-data-source-proxy: true
  config:
    type: nacos
    nacos:
      namespace:
      serverAddr: 127.0.0.1:8848
      group: SEATA_GROUP
      username: "nacos"
      password: "nacos"
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      group: SEATA_GROUP
      namespace:
      username: "nacos"
      password: "nacos"
```

### 第五步

运行你下载的nacos,并参考https://github.com/seata/seata/tree/develop/script/config-center 的config.txt并修改

文件在

![image-20210312170639465](https://xuemingde.com/pages/image/others/image-20210312170639465.png)

修改config.txt

```properties
service.vgroupMapping.my_test_tx_group=default
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.cj.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
store.db.user=username
store.db.password=password
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
```

运行仓库中提供的nacos脚本,将以上信息提交到nacos控制台,如果有需要更改,可直接通过控制台更改

进入文件夹下

![image-20210312171029260](https://xuemingde.com/pages/image/others/image-20210312171029260.png)

右键

![image-20210312171131042](https://xuemingde.com/pages/image/others/image-20210312171131042.png)

![image-20210312171216907](https://xuemingde.com/pages/image/others/image-20210312171216907.png)

 进入nacos查看

![image-20210312171537526](https://xuemingde.com/pages/image/others/image-20210312171537526.png)



### 第六步

更改server中的registry.conf

![image-20210312171654248](https://xuemingde.com/pages/image/others/image-20210312171654248.png)

![image-20210312171805716](https://xuemingde.com/pages/image/others/image-20210312171805716.png)

![image-20210312171822584](https://xuemingde.com/pages/image/others/image-20210312171822584.png)

### 第七步

运行seata-server

![image-20210312171941551](https://xuemingde.com/pages/image/others/image-20210312171941551.png)

![image-20210312172018047](https://xuemingde.com/pages/image/others/image-20210312172018047.png)