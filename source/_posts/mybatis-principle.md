---
title: mybatis架构中的一些概念
date: 2019-01-02 21:05:27
tags:
---

从三个框架中的具体处理来理解：mybatis，mybatis-spring，mybatis-spring-boot-autoconfigure

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

### 事务
Mybatis使用事务的前提配置：
```xml
<environments default="development">
  <environment id="development"> <!--定义的环境 ID-->
    <transactionManager type="JDBC"> <!--事务管理器-->
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED"> <!--数据源的配置-->
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```
> 参考：
> http://www.mybatis.org/mybatis-3/zh/configuration.html#environments

从入门代码分析整个事务的处理逻辑：

`1，SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);`

```java
/**
@see org.apache.ibatis.session.SqlSessionFactoryBuilder
**/
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      //调用build方法。
	  return build(parser.parse());
    } catch (Exception e) {
      ...
    } finally {
      ...
    }
}
  ...
public SqlSessionFactory build(Configuration config) {
	//创建默认的SqlSessionFactory。
    return new DefaultSqlSessionFactory(config);
}
```

`2，SqlSession session = sqlSessionFactory.openSession();`

```java
/**
@see org.apache.ibatis.session.defaults.DefaultSqlSessionFactory
**/
public SqlSession openSession() {
	return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
}
...
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
      final Environment environment = configuration.getEnvironment();
	  //根据配置获取事务管理器
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
	  //创建Transaction并设置DataSource、隔离级别、是否自动提交。
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
	  //创建Executor，它里面有commit、query等方法。
      final Executor executor = configuration.newExecutor(tx, execType);
	  //创建默认的SqlSession。并设置Executor等属性。
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      ...
    }
	...
}

private TransactionFactory getTransactionFactoryFromEnvironment(Environment environment) {
	/**
	根据transactionManager节点的type获取TransactionFactory。
	@see org.apache.ibatis.session.Configuration#Configuration()。
	**/
	
    if (environment == null || environment.getTransactionFactory() == null) {
      return new ManagedTransactionFactory();
    }
    return environment.getTransactionFactory();
}
/**
@see org.apache.ibatis.session.Configuration#newExecutor()
**/
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    ...
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      ...
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
      executor = new CachingExecutor(executor);
    }
    ...
    return executor;
}
```

`3，Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);`

```java
/**
SqlSession的SQL的具体执行过程是委托给Executor的。
**/
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
    try {
      MappedStatement ms = configuration.getMappedStatement(statement);
      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
    } catch (Exception e) {
      ...
    }
}

/**
默认开启缓存的话是CachingExecutor负责执行SQL的。
@see org.apache.ibatis.executor.CachingExecutor#query()
**/
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
      throws SQLException {
    Cache cache = ms.getCache();
    if (cache != null) {
      flushCacheIfRequired(ms);
      if (ms.isUseCache() && resultHandler == null) {
		...
		List<E> list = (List<E>) tcm.getObject(cache, key);
        if (list == null) {
          list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
          tcm.putObject(cache, key, list); // issue #578 and #116
        }
        return list;
      }
    }
    return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}

/**
查询数据库会委托给BaseExecutor和SimpleExecutor
@see org.apache.ibatis.executor.BaseExecutor#query()
@see org.apache.ibatis.executor.BaseExecutor#queryFromDatabase()
**/
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    ...
    List<E> list;
    try {
      queryStack++;
      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
      if (list != null) {
        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
      } else {
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
      }
    } finally {
      queryStack--;
    }
	...
    }
    return list;
}
/**
SimpleExecutor会调用JDBC的API。
@see org.apache.ibatis.executor.SimpleExecutor#doQuery()
**/
public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
      stmt = prepareStatement(handler, ms.getStatementLog());
      return handler.query(stmt, resultHandler);
    } finally {
      closeStatement(stmt);
    }
}

```

或者

`
4，BlogMapper mapper = session.getMapper(BlogMapper.class);
Blog blog = mapper.selectBlog(101);
`

```java
/**
使用Mapper的方式执行SQL会用到动态代理；最后还是会回到SqlSession中去执行SQL。
@see org.apache.ibatis.session.defaults.DefaultSqlSession#getMapper()
**/
public <T> T getMapper(Class<T> type) {
    return configuration.<T>getMapper(type, this);
}

/**
@see org.apache.ibatis.session.Configuration#getMapper()
**/
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
}

/**
@see org.apache.ibatis.binding.MapperRegistry#getMapper()
**/
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    ...
    try {
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      ...
    }
}

/**
@see org.apache.ibatis.binding.MapperProxyFactory
**/
protected T newInstance(MapperProxy<T> mapperProxy) {
	return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
}

public T newInstance(SqlSession sqlSession) {
	final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
	return newInstance(mapperProxy);
}

/**
Mapper代理类的详细执行过程可以看以下代码：
@see org.apache.ibatis.binding.MapperProxy
@see org.apache.ibatis.binding.MapperMethod#execute()
**/
```


## mybatis-spring

### SqlSessionFactoryBean
在基本的MyBatis中，SqlSessionFactory可以使用SqlSessionFactoryBuilder来创建
而在myBatis-spring中,则使用SqlSessionFactoryBean来替代。

### SqlSessionTemplate
SqlSessionTemplate是myBatis-spring的核心。这个类负责管理MyBatis的SqlSession，调用MyBatis的SQL方法。
SqlSessionTemplate是线程安全的，可以被多个DAO所共享使用；SqlSessionTemplate将会保证使用的SqlSession是和当前Spring的事务相关的。

### Mapper
为了代替手工使用SqlSessionDaoSupport或SqlSessionTemplate编写数据访问对象(DAO)的代码，myBatis-spring提供了一个动态代理的实现：MapperFactoryBean。
这个类可以直接注入Mapper接口到service层bean中。当使用Mapper时，直接调用就可以了，不需要编写任何实现的代码，因为myBatis-spring将会为你创建代理。

MapperFactoryBean创建的代理控制打开和关闭session，还能转换任意的异常到Spring的DataAccessException异常中。
此外,如果要加入到一个已经存在活动事务中，代理将会开启一个新的 Spring 事务。

### 事务
mybatis-spring使用事务的前提配置：

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <!--将DataSource注入到SqlSessionFactoryBean中-->
  <property name="dataSource" ref="dataSource" />
  <!--指定mybatis的Mapper对应的XML文件的位置。-->
  <!--还有一种配置方式是在mybatis的XML配置文件中使用<mappers>部分来指定类路径。-->
  <property name="mapperLocations" value="classpath*:sample/config/mappers/**/*.xml" />
</bean>

<!--开启Spring的事务处理，配置之后，就可以在Spring中用@Transactional注解和AOP样式的配置来管理事务-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource" />
</bean>

<!--配置SqlSessionTemplate，代替mybatis默认实现的DefaultSqlSession-->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>

<!--在DAO中注入SqlSession-SqlSessionTemplate，这样DAO中就可以直接使用SqlSession了。-->
<bean id="userDao" class="org.xxx.xxx.dao.UserDaoImpl">
  <property name="sqlSession" ref="sqlSession" />
</bean>

<!--如果不想以上面的DAO的方式显式调用SqlSession，可以使用Mapper接口的方式访问数据。-->
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
  <!--MapperFactoryBean会为UserMapper创建代理。-->
  <property name="mapperInterface" value="org.xxx.xxx.mapper.UserMapper" />
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>

<!--如果不想以上面XML的方式配置每个Mapper，可以使用MapperScannerConfigurer；它将会查找类路径下的Mapper并自动将它们创建成MapperFactoryBean-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value="org.xxx.xx.mapper" />
</bean>
```

接下来看看以上各个类的部分源码：
- SqlSessionFactoryBean创建SqlSessionFactory的过程：

```java
/**
@see org.mybatis.spring.SqlSessionFactoryBean#buildSqlSessionFactory()
**/
protected SqlSessionFactory buildSqlSessionFactory() throws IOException {

    Configuration configuration;

    XMLConfigBuilder xmlConfigBuilder = null;
    
	//主要是读取配置文件的内容
	...

	//默认用SpringManagedTransactionFactory代替mybatis的TransactionFactory
    if (this.transactionFactory == null) {
      this.transactionFactory = new SpringManagedTransactionFactory();
    }
	
	...
    configuration.setEnvironment(new Environment(this.environment, this.transactionFactory, this.dataSource));
	...
	
    if (!isEmpty(this.mapperLocations)) {
	  ...
      for (Resource mapperLocation : this.mapperLocations) {
        XMLMapperBuilder xmlMapperBuilder = new XMLMapperBuilder(mapperLocation.getInputStream(),
              configuration, mapperLocation.toString(), configuration.getSqlFragments());
          xmlMapperBuilder.parse();
	  }
	  ...
    }
	
	//使用mybatis的SqlSessionFactoryBuilder创建SqlSessionFactory。
    return this.sqlSessionFactoryBuilder.build(configuration);
}
```

- SqlSessionTemplate代替SqlSession的过程：

```java
/**
SqlSessionTemplate在实例化Bean时会创建SqlSession的Proxy。
@see org.mybatis.spring.SqlSessionTemplate
**/
public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType,
      PersistenceExceptionTranslator exceptionTranslator) {
	
	...
	
    this.sqlSessionFactory = sqlSessionFactory;
    this.executorType = executorType;
    this.exceptionTranslator = exceptionTranslator;
	//使用JDK的动态代理，创建SqlSession的Proxy。
	//SqlSessionTemplate中各种访问数据库的方法都是由SqlSessionProxy负责执行的。
    this.sqlSessionProxy = (SqlSession) newProxyInstance(
        SqlSessionFactory.class.getClassLoader(),
        new Class[] { SqlSession.class },
        new SqlSessionInterceptor());
  }
  
/**
这是SqlSession的代理实现类
@see org.mybatis.spring.SqlSessionTemplate.SqlSessionInterceptor
**/

private class SqlSessionInterceptor implements InvocationHandler {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	  /**
		静态导入的@see org.mybatis.spring.SqlSessionUtils.getSqlSession 方法。
		它的作用是:首先从当前事务的线程中获取SqlSession；如果有，直接返回SqlSession；
		如果没有，则用SqlSessionFactory创建一个，并绑定到当前事务的线程中。
	  **/
      SqlSession sqlSession = getSqlSession(
          SqlSessionTemplate.this.sqlSessionFactory,
          SqlSessionTemplate.this.executorType,
          SqlSessionTemplate.this.exceptionTranslator);
      try {
        Object result = method.invoke(sqlSession, args);
        ...
        return result;
      } catch (Throwable t) {
        ...
      } finally {
        if (sqlSession != null) {
          closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
        }
      }
    }
}
```

- MapperFactoryBean生成Mapper代理的过程：

```java
/**
该类通过继承SqlSessionDaoSupport，间接实现了InitializingBean。
@see org.mybatis.spring.mapper.MapperFactoryBean#checkDaoConfig
**/
protected void checkDaoConfig() {
    super.checkDaoConfig();
	...
    Configuration configuration = getSqlSession().getConfiguration();
    if (this.addToConfig && !configuration.hasMapper(this.mapperInterface)) {
      try {
		/**
		注册Mapper接口，并创建Mapper对应的MapperProxyFactory对象。
		**/
        configuration.addMapper(this.mapperInterface);
      } catch (Exception e) {
        ...
      }
    }
}
```

- MapperScannerConfigurer扫描并注册Mapper的过程：

```java
/**

@see org.mybatis.spring.mapper.MapperScannerConfigurer#postProcessBeanDefinitionRegistry()
**/
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    if (this.processPropertyPlaceHolders) {
      processPropertyPlaceHolders();
    }

    ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
    //设置配置文件中的各种信息。
	scanner.setAddToConfig(this.addToConfig);
    ...
    scanner.registerFilters();
    scanner.scan(StringUtils.tokenizeToStringArray(this.basePackage, ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS));
}

/**
@see org.mybatis.spring.mapper.ClassPathMapperScanner#doScan()
**/

public Set<BeanDefinitionHolder> doScan(String... basePackages) {
    Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);

    if (beanDefinitions.isEmpty()) {
      LOGGER.warn(() -> "No MyBatis mapper was found in '" + Arrays.toString(basePackages) + "' package. Please check your configuration.");
    } else {
      processBeanDefinitions(beanDefinitions);
    }

    return beanDefinitions;
}

private void processBeanDefinitions(Set<BeanDefinitionHolder> beanDefinitions) {
    GenericBeanDefinition definition;
    for (BeanDefinitionHolder holder : beanDefinitions) {
      definition = (GenericBeanDefinition) holder.getBeanDefinition();
	  ...
	  //将Mapper转换成MapperFactoryBean
      definition.setBeanClass(this.mapperFactoryBean.getClass());
	  ...
	  //将SqlSessionFactory、SqlSessionTemplate等设置到BeanDefinition，也即是MapperFactoryBean中。
	  definition.getPropertyValues().add("sqlSessionFactory", this.sqlSessionFactory);
	  ...
      definition.getPropertyValues().add("sqlSessionTemplate", this.sqlSessionTemplate);

    }
}

```

## mybatis-spring-boot-autoconfigure
它的主要功能是自动装配`SqlSessionFactory`、`SqlSessionTemplate`以及用`AutoConfiguredMapperScannerRegistrar`代替`MapperScannerConfigurer`的功能。

*记录一个疑问：在auto configure的方式下，MapperFactoryBean中的SqlSessionTemplate是怎么注入进去的？*