---
layout: post
read_time: true
show_date: true
title: "Spring-boot data JPA(一)建库与一对多、多对多关系"
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

选取MySQL为例，引入mysql驱动

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>

同时在application.yml中做如下配置

    spring:
      datasource:
        url: jdbc:mysql://127.0.0.1:3306/database?serverTimezone = GMT%2B8 & useUnicode=true #最后一项为使用utf-8编码
        username: root
        password: root
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

JPA可以协助使用者依据实体类创建表，这要求使用者提供标注，以下为实例

    @Entity //实体
    @Table(name = "user")//JPA会依据name建表读表
    public class User{
      @Id//标识主键
      @GeneratedValue(strategy = GenerationType.IDENTITY)//主键自增策略
      private Integer id;
      @Column(name = "name", unique = true, nullable = false, length = 64)//name属性对应表中的属性，unique，nullable分别对应属性是否唯一，是否为空（默认为否），length则代表存储长度
      private String username;
      //省略空参构造，与get/set函数
    }

为避免空参构造与get/set的繁琐，提高代码的简洁性，可以引入lombok

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <scope>provided</scope>
    </dependency>

lombok提供以下注释

| 注释                     | 作用域及作用                                                 |
| ------------------------ | ------------------------------------------------------------ |
| @Setter                  | 注解在类或字段，注解在类时为所有字段生成setter方法，注解在字段上时只为该字段生成setter方法。 |
| @Getter                  | 注解在类或字段，注解在类时为所有字段生成getter方法，注解在字段上时只为该字段生成getter方法 |
| @ToString                | 注解在类，添加toString方法。                                 |
| @EqualsAndHashCode       | 注解在类，生成hashCode和equals方法。                         |
| @NoArgsConstructor       | 注解在类，生成无参的构造方法。                               |
| @RequiredArgsConstructor | 注解在类，为类中需要特殊处理的字段生成构造方法，比如final和被@NonNull注解的字段。 |
| @AllArgsConstructor      | 注解在类，生成包含类中所有字段的构造方法。                   |
| @Data                    | 注解在类，生成setter/getter、equals、canEqual、hashCode、toString方法，如为final属性，则不会为该属性生成setter方法 |
| @Slf4j                   | 注解在类，生成log变量，严格意义来说是常量。 |

根据我们的需求，重新构造User

    @Data
    @NoArgsConstructor
    @Entity
    @Table(name = "user")
    public class User{
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Integer userId;
      @Column(name = "name", unique = true, nullable = false, length = 64)
      private String username;
    }

可以为User添加一对多关系Admin，一个Admin管理多个User

    @Data
    @NoArgsConstructor
    @Entity
    @Table(name = "admin")
    public class Admin{
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Integer adminId;
      @OneToMany(targetEntity = User.class, cascade = {CascadeType.PERSIST, CascadeType.MERGE ,CascadeType.REFRESH})
      //targetEntity指向实体类，cascade代表级联方式
      @JoinColumn(name = "adminId")
      //name代表作为外键的属性
      private Collection<User> users;
    }

同时User中添加以下一行

    @ManyToOne(cascade = {CascadeType.PERSIST, CascadeType.MERGE,CascadeType.REFRESH})
    private Admin adminId;

可以为User添加多对多关系Role，在JPA中，多对多的双方并不是完全对等的，通常分为支配方与被支配方
选择User作为支配方，添加代码

    @ManyToMany(targetEntity = Role.class, cascade = {CascadeType.PERSIST, CascadeType.MERGE,CascadeType.REFRESH})
    @JoinTable(name = "role2tag",//多对多关系在数据库中需要创建新表，这一行是表名
            joinColumns = {@JoinColumn(name = "userId", referencedColumnName = "userId")},//两个属性分别支配方被引用的属性，以及改属性在role2tag表中的名称
            inverseJoinColumns = {@JoinColumn(name = "roleId", referencedColumnName = "roleId")}//两个属性分别被支配方被引用的属性，以及改属性在role2tag表中的名称
    )
    private Collection<Role> roles;

同时创建实体类Role

    @Data
    @NoArgsConstructor
    @Entity
    @Table(name = "admin")
    public class Role{
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Integer roleId;
      @ManyToMany(targetEntity = User.class, mappedBy = "roles", cascade = {CascadeType.PERSIST, CascadeType.MERGE,CascadeType.REFRESH})
      //mappedBy对应支配方引用该实体的属性，即roles
      private Collection<User> users;
    }

JPA的级联属性如下所示：

| 级联属性 | 作用     |
| -------- | -------- |
| PERSIST  | 级联保存 |
| MERGE    | 级联更新 |
| REMOVE   | 级联删除 |
| REFRESH  | 级联获取 |
| DETACH   | 级联移除 |
| ALL     | 以上全部 |

REFRESH、DETACH中的级联对应服务器从数据库中获取数据，服务器从内存中移除数据的过程，这涉及到JPA的运作过程：
当JPA查询到有关联关系的存在时，不会立即获取被关联的对象，而是在被调用时再获取
