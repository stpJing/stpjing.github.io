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
description: "JPA只是一个简化对象关系映射来管理Java应用程序中的关系数据的规范。 它提供了一个平台，可以直接使用对象而不是使用SQL语句。"
---
JPA只是一个简化对象关系映射来管理Java应用程序中的关系数据的规范。 它提供了一个平台，可以直接使用对象而不是使用SQL语句。

对于SpringBoot而言，整合JPA是容易的，只需要在maven中引入以下依赖：
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
