---
title: MyBatis-Plus引入p6spy打印执行SQL
date: 2022-09-23 16:39:53
tags: [Java,MyBatis-Plus,日常随笔]
categories: [Java,日常随笔]
---
引入p6spy方便查看执行日志，`MyBatis-Plus3.2.1以上`使用

[官方文档地址](https://baomidou.com/pages/833fab/)
#### 1、引入依赖
```java
<dependency>
  <groupId>p6spy</groupId>
  <artifactId>p6spy</artifactId>
  <version>最新版本</version>
</dependency>
```
具体版本号，可以去maven仓库去查询。

[地址](https://mvnrepository.com/artifact/p6spy/p6spy)
#### 2、application.yml 配置
```yaml
spring:
  datasource:
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver
    url: jdbc:p6spy:h2:mem:test
    ...
```
#### 3、spy.properties 配置
```text
#3.2.1以上使用
modulelist=com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory,com.p6spy.engine.outage.P6OutageFactory
#3.2.1以下使用或者不配置
#modulelist=com.p6spy.engine.logging.P6LogFactory,com.p6spy.engine.outage.P6OutageFactory
# 自定义日志打印
logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger
#日志输出到控制台
appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger
# 使用日志系统记录 sql
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
# 设置 p6spy driver 代理
deregisterdrivers=true
# 取消JDBC URL前缀
useprefix=true
# 配置记录 Log 例外,可去掉的结果集有error,info,batch,debug,statement,commit,rollback,result,resultset.
excludecategories=info,debug,result,commit,resultset
# 日期格式
dateformat=yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
#driverlist=org.h2.Driver
# 是否开启慢SQL记录
outagedetection=true
# 慢SQL记录标准 2 秒
outagedetectioninterval=2
```
#### 4、执行效果：
```text
 Consume Time：3 ms 2022-09-23 14:45:47
 Execute SQL：SELECT t1.id source_id, t1.source_name, t2.id table_id, t2.table_name FROM `st_source` t1 LEFT JOIN st_table t2 ON t1.source_hash_code = t2.source_hash_code WHERE t1.is_delete = 0 AND t2.is_delete = 0
```
