# Mybatis源码（3）

- ## 入口语句(通过SqlSessionFactory创建Sqlsession对象并通过Sqlsession获取Mapper接口)

  ```java
  SqlSession session = factory.openSession();//创建Sqlsession对象
  UserMapper mapper = session.getMapper(UserMapper.class);
  ```

- ## opensession方法

  ```java
  private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
      Transaction tx = null;
      try {
        final Environment environment = configuration.getEnvironment();//获取environment名称
        final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);//获取该环境下的TransactionFactory，如果该环境下没有指定事务工厂则采用ManagedTransactionFactory
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);//使用环境工厂创建事务
        final Executor executor = configuration.newExecutor(tx, execType);//根据执行器类型创建执行对象
        return new DefaultSqlSession(configuration, executor, autoCommit);//创建DefaultSqlSession（只是简单赋值）
      } catch (Exception e) {
        closeTransaction(tx); // may have fetched a connection so lets call close()
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
      } finally {
        ErrorContext.instance().reset();
      }
    }
  ```
  
  ### newExecutor方法
  
  ```java
  public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
      executorType = executorType == null ? defaultExecutorType : executorType;
      executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
      Executor executor;
      if (ExecutorType.BATCH == executorType) {//根据不同类型创建执行器对象
        executor = new BatchExecutor(this, transaction);
      } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
      } else {
        executor = new SimpleExecutor(this, transaction);
      }
      if (cacheEnabled) {
        executor = new CachingExecutor(executor);//如果开启了缓存则创建缓存执行器
      }
      executor = (Executor) interceptorChain.pluginAll(executor);//通过插件进行加强
      return executor;
    }
  ```
  
- ## SqlSession的getMapper方法

  ```java
  public <T> T getMapper(Class<T> type) {
      return configuration.getMapper(type, this);
  }
  //configuration的getMapper方法
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
      return mapperRegistry.getMapper(type, sqlSession);
  }
  //mapperRegistry的getMapper方法
  //mapperRegistry的成员变量
  private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();
  
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
      final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);//从knownMappers中获取mapperProxyFactory
      if (mapperProxyFactory == null) {
          throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
      }
      try {
          return mapperProxyFactory.newInstance(sqlSession);
      } catch (Exception e) {
          throw new BindingException("Error getting mapper instance. Cause: " + e, e);
      }
  }
  ```

  