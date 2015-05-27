---
layout: post 
title: 使用flyway对应用的数据库版本控制
description: 我们可以想象，多位开发人员，多个数据库环境。偶尔就出现：哎呀！集成环境的数据库忘记更新了。推荐使用flyway进行应用程序的数据库版本控制。
category: blog
---
![image]({{ site.url }}/images/why_db_migraion_tools.png)

## 为什么要使用DB migration tools

我们可以想象，多位开发人员，多个数据库环境。  
偶尔就出现：哎呀！集成环境的数据库忘记更新了。  

## 为什么推荐flyway

### 简单，好用
先前有用过mybatis中的migration，但经常出现莫名其名的异常，并且也不支持多条SQL在1个文件中。    
最开始在使用flyway时，没有downgrade。有些觉得奇怪，像mybatis的migration和rails中，都会支持。   
后来想想，其实对数据库的downgrade真是要甚用（最好不用），会陷入麻烦，不清楚真实的版本变化。宁可重新写个script downgrade。

### 支持java调用及spring集成
这就能在应用程序中，直接进行管理。  
我采用的方式，是根据web应用在不同的环境启动时，进行不同环境 db migration.  
在往集成发布时，随tomcat执行数据库的版本同步。    
具体的使用，[flyway官方文档](http://flywaydb.org/getstarted/)详细简洁，几下就轻松搞定。

### 跟spring集成
这也是非常非常的简化

```xml

<bean id="flyway" class="org.flywaydb.core.Flyway" depends-on="dataSource_flyway" lazy-init="false">
    <property name="dataSource" ref="dataSource_flyway"/>
</bean>

<bean id="jFlyway" class="net.kkuang.flyway.DbMigration" lazy-init="false" depends-on="flyway">
    <property name="flyway" ref="flyway"/>
</bean>
```

```java
/**
 * 执行DbMigration
 * 当应用服务启动时会自动执行
 *
 * User: 闷骚乔巴
 * Date: 2014-12-02
 */
public class DbMigration {

    private Log log = LogFactory.getLog(DbMigration.class);

    private Flyway flyway;

    @PostConstruct
    public void run() {
        log.info("[Start] DbMigration run .. ");

        flyway.migrate();

        log.info("[End] DbMigration run .. ");
    }

    public void setFlyway(Flyway flyway) {
        this.flyway = flyway;
    }

}

```

## SQL Script 命名

### script 目录
flyway执行时，默认读取的目录是`classpath:/db/migration`  
我们项目中就放在`/resource/db/migration`  

### 文件名 
![image]({{ site.url }}/images/SqlMigrationNaming.png)  
该文件名由：   
* prefix: default: V (大写哦)  
* version: 版本号，也可以使用大小版本组合的方式，小版本号用单`_`区分   
* separator: 分隔符，双下划线`__`    
* description: 描述（你懂得，必须要有意义）   
* suffix: 后缀 default: `.sql`    

例如：`V1_1__create_table_test.sql`

**再也不用担心，各环境的数据库不一致了。**
