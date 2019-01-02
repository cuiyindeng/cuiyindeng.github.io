---
title: mybatis架构中的一些概念
date: 2019-01-02 21:05:27
tags:
---

从下面三个框架讲解：mybatis，mybatis-spring，mybatis-spring-boot-autoconfigure

<!-- more -->

## mybatis

### mybatis中基础对象的scope和lifecycle
理解不同作用域和生命周期类是至关重要的，因为错误的使用会导致并发问题。

#### 1，SqlSessionFactoryBuilder

这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。
因此 SqlSessionFactoryBuilder 实例的最佳作用域是method作用域。

#### 2，SqlSessionFactory
SqlSessionFactory 一旦被创建就应该在 application 的运行期间一直存在，没有任何理由对它进行清除或重建。
使用 SqlSessionFactory 的最佳实践是在 application 运行期间不要重复创建多次；SqlSessionFactory 的最佳作用域是 application 作用域

#### 3，SqlSession
每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，
所以它的最佳的作用域是 request 或者 method 作用域。

SqlSession相当于一个JDBC的Connection对象，也可以说它的生命周期是在请求数据库处理事务的过程中，
它的长期存在对数据库连接池影响很大，应该及时close。
它存活于一个应用的请求和操作，可以执行多条SQL，保证事务的一致性。

#### 4，Mapper Instances
映射器是用来绑定映射的语句的接口。映射器接口的实例是从 SqlSession 中获得的。
因此，任何映射器实例的最大作用域是和请求它们的 SqlSession 相同的，并且映射器实例的最佳作用域是method作用域。
也就是说，映射器实例应该在调用它们的方法中被请求，用过之后即可废弃，并不需要显式地关闭映射器实例。


## mybatis-spring

### SqlSessionFactoryBean
在基本的 MyBatis 中,session 工厂可以使用 SqlSessionFactoryBuilder 来创建。而在 MyBatis-Spring 中,则使用 SqlSessionFactoryBean 来替代。


### 事务

### SqlSessionTemplate
SqlSessionTemplate 是 MyBatis-Spring 的核心。这个类负责管理 MyBatis 的 SqlSession, 调用 MyBatis 的 SQL 方法。
SqlSessionTemplate 是线程安全的, 可以被多个 DAO 所共享使用; SqlSessionTemplate 将会保证使用的 SqlSession 是和当前 Spring 的事务相关的。

### Mapper
为了代替手工使用 SqlSessionDaoSupport 或 SqlSessionTemplate 编写数据访问对象(DAO)的代码，MyBatis-Spring 提供了一个动态代理的实现:MapperFactoryBean。
这个类可以让你直接注入Mapper接口到你的 service 层 bean 中。当使用Mapper时，直接调用就可以了，不需要编写任何实现的代码，因为 MyBatis-Spring 将会为你创建代理。
MapperFactoryBean 创建的代理控制开放和关闭 session，转换任意的异常到 Spring 的 DataAccessException 异常中。
此外,如果要加入到一个已经存在活动事务中，代理将会开启一个新的 Spring 事务。

## mybatis-spring-boot-autoconfigure

### 自动配置
