---
layout: post
read_time: true
show_date: true
title: "JPA 探究"
date: 2022-05-10
tags: [Spring-boot, JPA, SQL注入]
category: opinion
author: 荆凉凉
img: posts/20220510/title.png
description: "JPA只是一个简化对象关系映射来管理Java应用程序中的关系数据的规范。 它提供了一个平台，可以直接使用对象而不是使用SQL语句。
对于SpringBoot而言，整合JPA是容易的，只需要在maven中引入以下依赖："
---
JPA只是一个简化对象关系映射来管理Java应用程序中的关系数据的规范。 它提供了一个平台，可以直接使用对象而不是使用SQL语句。

对于SpringBoot而言，整合JPA是容易的，只需要在maven中引入以下依赖：

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

选取MySQL为例，引入mysql驱动

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>

同时在application.yml中做如下配置

    spring:
      datasource:
        url: jdbc:mysql://{{数据库IP+端口（一般为3306）}}/{{数据库名}}?serverTimezone = {{时区}} & useUnicode=true #最后一项为使用utf-8编码
        username: {{用户名}}
        password: {{密码}}
        driver-class-name: com.mysql.cj.jdbc.Driver
      jpa:
        database: MySQL
        database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
        show-sql: true #是否自动显示sql语句
        hibernate:
         ddl-auto: update

需要注意的是，ddl-auto有五个属性

| ddl-auto    | 功能                                                         |
| ----------- | ------------------------------------------------------------ |
| create      | 程序每次时，检查标注的实体类，并依据实体类重新建表，会导致数据丢失 |
| create-drop | 程序运行时，检查标注的实体类，并依据实体类重新建表，并在程序结束时销毁表 |
| update      | 程序运行时，检查标注的实体类，如果没有表就建表，如果当前表内字段属性与实体类中不符，则会更新表 |
| validate    | 程序运行时，检查标注的实体类，如果当前表内字段属性与实体类中不符，或不存在对应的表，则会更新表 |
| none        | 禁用                                                         |

从这一点来看
