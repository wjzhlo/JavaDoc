## 基本

ShardingSphere-JDBC

ShardingSphere-Proxy

ShardingSphere-Scaling --> 用于数据扩容

ShardingSphere-UI --> 界面管理





### ShardingSphere-JDBC

ShardingSphere-JDBC 能够让我们自由访问不同库的不同表，以 jar 包形式嵌入

`ShadingSphere-JDBC 更偏向应用端，即分库分表逻辑在应用端实现`

![image-20210602223856396](E:\lincya\doc\ShardingSphere\ShardingSphere-JDBC.assets\image-20210602223856396.png)





### ShardingSphere-Proxy

ShardingSphere-Proxy 会在整个数据库代理层伪装成一个 mysql，应用直接连接 ShardingSphere-proxy 形成的代理数据库，从而 proxy 实现分库分表

`ShardingSphere 更偏向服务端，即应用端无需管分库分表的逻辑，分库分表由代理完成`

![image-20210602223835677](E:\lincya\doc\ShardingSphere\ShardingSphere-JDBC.assets\image-20210602223835677.png)



### S-JDBC 和 S-Proxy 的区别

![image-20210602225427037](E:\lincya\doc\ShardingSphere\ShardingSphere-JDBC.assets\image-20210602225427037.png)

![image-20210602230631715](E:\lincya\doc\ShardingSphere\ShardingSphere-JDBC.assets\image-20210602230631715.png)

#### S-Proxy

Proxy 的方式数据都依赖于 S-Proxy 来处理，而 S-Proxy 当前仅支持 MySQL 和 PostGreSQL，但 Proxy 的方式对业务侵入低，业务层不用管如何分库分表的



#### S-JDBC

S-JDBC 是对数据库的拓展，即 jdbc 可以连什么数据库，S-JDBC 就可以连什么数据库



### Registry-Center

- 集中配置
- 快速伸缩容



## 概念

### 作用

分库分表和读写分离，通过分库分表的方式来解决由于数据量大导致数据库性能降低的问题

![image-20210602231702835](E:\lincya\doc\ShardingSphere\ShardingSphere-JDBC.assets\image-20210602231702835.png)

- 取模
  - 不利于缩容和扩容
- 按照范围分片
  - 好扩容和缩容，但数据分布不够均匀



### 缺点

- 事务一致性问题
- 跨节点关联查询
  - 分库后，表被分散到了不同的数据库，无法进行关联查询，需要将关联查询拆分成多次查询，然后将获得的结果进行拼装
- 跨界点分页、排序函数
  - 当跨节点跨库进行查询时，limit分页，order by排序问题，就需要先在不同的分片节点中将数据进行排序并返回，然后将不同分片返回的结果进行汇总和再次排序
- 主键避重问题
  - 在分库分表环境下，不能够使用自增长的主键id，需要单独设计全局主键，避免跨库重复问题
- 公共表处理
  - 在实际场景中，参数表、数据字典表等都是数据量较小，变动少，而且属于高频联合查询的依赖表，这类表一般都在每个数据库都保存一份，并且所有对公共表的操作都要分发到所有的分库去执行
- 运维工作量
  - 分库分表会提升运维成本



### 什么时候需要分库分表

阿里巴巴开发手册中写到，当 mysql 单表记录如果达到 500w，或是单表容量达到 2GB，就建议分库分表













## 主要产品对比

ShardingSphere、MyCat、DBLE

`ShardingSphere 提供了标准化的数据分片、分布式事务和数据库治理功能`

![image-20210602233624591](E:\lincya\doc\ShardingSphere\ShardingSphere-JDBC.assets\image-20210602233624591.png)



## 配置

```properties
spring.application.name=shardingsphere-xi-demo

######## 定义数据源
spring.shardingsphere.datasource.names=db0,db1
# 配置具体的某个数据源
spring.shardingsphere.datasource.db0.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.db0.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.db0.url=jdbc:mysql://www.gtom.top:3306/shardingsphere_test_0
spring.shardingsphere.datasource.db0.username=root
spring.shardingsphere.datasource.db0.password=root
# 配置具体的某个数据源
spring.shardingsphere.datasource.db1.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.db1.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.db1.url=jdbc:mysql://www.gtom.top:3306/shardingsphere_test_1
spring.shardingsphere.datasource.db1.username=root
spring.shardingsphere.datasource.db1.password=root

######## 配置默认的分库策略
spring.shardingsphere.sharding.tables.course.database-strategy.inline.sharding-column=cid
spring.shardingsphere.sharding.tables.course.database-strategy.inline.algorithm-expression=db$->{cid%2}

######## 定义逻辑表
# 配置对应的实际表
spring.shardingsphere.sharding.tables.course.actual-data-nodes=db$->{0..1}.course_$->{1..2}
# 配置主键生成策略
spring.shardingsphere.sharding.tables.course.key-generator.column=cid
spring.shardingsphere.sharding.tables.course.key-generator.type=SNOWFLAKE
spring.shardingsphere.sharding.tables.course.key-generator.props.worker.id=1
# 设置分片键
spring.shardingsphere.sharding.tables.course.table-strategy.inline.sharding-column=user_id
# 设置分片算法,等同按奇偶区分开
spring.shardingsphere.sharding.tables.course.table-strategy.inline.algorithm-expression=course_$->{user_id%2+1}

# 设置展示 sql 语句
spring.shardingsphere.props.sql.show=true
# bean 定义允许覆盖
spring.main.allow-bean-definition-overriding=true
```

