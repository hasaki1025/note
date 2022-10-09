# Mybatis源码（4）

- ## 入口语句(Mapper执行SQL语句)

  ```java
  mapper.selectByIdWithRole(user);
  ```

- ## selectByIdWithRole方法（Mapper已经通过代理类进行了加强，这里执行的是加强后的代理类的Invoke方法）

  ```java
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      try {
          if (Object.class.equals(method.getDeclaringClass())) {
              return method.invoke(this, args);//对于Mapper接口之后的实现类的原生方法（非Sql方法）进行放行
          } else {
              return cachedInvoker(method).invoke(proxy, method, args, sqlSession);//通过cachedInvoker获取MethodInvoke类，由MethodInvoke类执行invoke方法（Mapper方法）
          }
      } catch (Throwable t) {
          throw ExceptionUtil.unwrapThrowable(t);
      }
  }
  ```

  - ###  cachedInvoker(method)方法

    ```java
    //MapperProxy中的成员变量
    private final Map<Method, MapperMethodInvoker> methodCache;
    
    private MapperMethodInvoker cachedInvoker(Method method) throws Throwable {
        try {
            return methodCache.computeIfAbsent(method, m -> {//computeIfAbsent：如果含有该method方法则直接返回对应的methodInvoker，否则通过指定接口创建该methodInvoker
                if (m.isDefault()) {//该方法是否是default修饰
                    try {
                        if (privateLookupInMethod == null) {//暂时不知道有上面用
                            return new DefaultMethodInvoker(getMethodHandleJava8(method));
                        } else {
                            return new DefaultMethodInvoker(getMethodHandleJava9(method));
                        }
                    } catch (IllegalAccessException | InstantiationException | InvocationTargetException
                             | NoSuchMethodException e) {
                        throw new RuntimeException(e);
                    }
                } else {//非default修饰的方法直接创建PlainMethodInvoker
                    return new PlainMethodInvoker(new MapperMethod(mapperInterface, method, sqlSession.getConfiguration()));
                }
            });
        } catch (RuntimeException re) {
            Throwable cause = re.getCause();
            throw cause == null ? re : cause;
        }
    }
    ```

    - #### MapperMethod的构造方法

      ```java
      public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
          this.command = new SqlCommand(config, mapperInterface, method);
          this.method = new MethodSignature(config, mapperInterface, method);
      }
      ```

      - ##### SqlCommand(用于表示SQL语句的ID和类型)的构造方法

        ```java
        public SqlCommand(Configuration configuration, Class<?> mapperInterface, Method method) {
            final String methodName = method.getName();//获取Mapper方法名称
            final Class<?> declaringClass = method.getDeclaringClass();//获取Mapper方法调用类
            MappedStatement ms = resolveMappedStatement(mapperInterface, methodName, declaringClass,
                                                        configuration);//从Configuration中获取MappedStatement
            if (ms == null) {//如果没有该语句则判断是否含有Flush注解，如果不含有Flush注解则直接抛出异常
                if (method.getAnnotation(Flush.class) != null) {
                    name = null;
                    type = SqlCommandType.FLUSH;
                } else {
                    throw new BindingException("Invalid bound statement (not found): "
                                               + mapperInterface.getName() + "." + methodName);
                }
            } else {//成员变量赋值
                name = ms.getId();
                type = ms.getSqlCommandType();
                if (type == SqlCommandType.UNKNOWN) {
                    throw new BindingException("Unknown execution method for: " + name);
                }
            }
        }
        ```

        - ###### SqlCommand的resolveMappedStatement方法

          ```java
          private MappedStatement resolveMappedStatement(Class<?> mapperInterface, String methodName,
                                                         Class<?> declaringClass, Configuration configuration) {
              String statementId = mapperInterface.getName() + "." + methodName;//statementId只是通过简单的拼接，为接口全路径名称+方法名称而Mapper进行注册时的接口可能是当前参数（mapperInterface）的父类所以采用递归查找该方法
              if (configuration.hasStatement(statementId)) {
                  return configuration.getMappedStatement(statementId);//从configuration中的mappedStatements获取MappedStatement，期间会检查是否还有未完成解析的MapperStatement，缓存引用、缓存等如果有则进行解析
              } else if (mapperInterface.equals(declaringClass)) {//如果方法调用者是Mapper接口本身则返回空
                  return null;
              }
              for (Class<?> superInterface : mapperInterface.getInterfaces()) {
                  if (declaringClass.isAssignableFrom(superInterface)) {//查看是否方法调用者实现了父类的接口
                      MappedStatement ms = resolveMappedStatement(superInterface, methodName,
                                                                  declaringClass, configuration);//若实现了父类的接口则进入递归将父类作为参数传入直到可以从configuration中找到该方法
                      if (ms != null) {//找到了就直接返回,没有找到则继续从父类中查找
                          return ms;
                      }
                  }
              }
              return null;
          }
          }
          ```

          -  configuration的getMappedStatement方法及其后续方法

            ```java
            public MappedStatement getMappedStatement(String id) {
                return this.getMappedStatement(id, true);
              }
            
              public MappedStatement getMappedStatement(String id, boolean validateIncompleteStatements) {
                if (validateIncompleteStatements) {//是否需要校验是否还有MapperStatement没有解析
                  buildAllStatements();//解析Mapper.xml文件
                }
                return mappedStatements.get(id);//从MappedStatements中获取MappedStatement
              }
            //buildAllStatements方法
            
            protected void buildAllStatements() {
                parsePendingResultMaps();//解析所有未解析的ResultMap并清空incompleteCacheRefs
                if (!incompleteCacheRefs.isEmpty()) {
                    synchronized (incompleteCacheRefs) {//解析所有未解析的缓冲引用
                        incompleteCacheRefs.removeIf(x -> x.resolveCacheRef() != null);
                    }
                }
                if (!incompleteStatements.isEmpty()) {//incompleteStatements是一个集合，该集合中保存了所有SQL标签的所有内容，如果此时incompleteStatements不为空则解析所有SQL标签，该if块作用是解析所有SQL标签并清空incompleteStatements
                    synchronized (incompleteStatements) {
                        incompleteStatements.removeIf(x -> {//removeIf：循环遍历该集合中的所有节点，根据接口括号内的方法返回值确认该节点是否需要删除,这里是选择全部删除
                            x.parseStatementNode();//解析Mapper.xml文件中一条SQL语句标签中的所有信息
                            return true;//删除该节点
                        });
                    }
                }
                if (!incompleteMethods.isEmpty()) {//解析所有MapperMethod方法（大体根据注解）并清空incompleteMethods
                    synchronized (incompleteMethods) {
                        incompleteMethods.removeIf(x -> {
                            x.resolve();
                            return true;
                        });
                    }
                }
            }
            ```

      - ##### MethodSignature的构造方法

        ```java
        //MethodSignature主要用于表示Mapper方法的返回值，参数的位置，参数解析器等信息
        public MethodSignature(Configuration configuration, Class<?> mapperInterface, Method method) {
            Type resolvedReturnType = TypeParameterResolver.resolveReturnType(method, mapperInterface);//解析返回参数类型（主要针对泛型设计），根据泛型类型返回ParameterizedType、GenericArrayType、TypeVariable、type类型
            if (resolvedReturnType instanceof Class<?>) {//将resolvedReturnType转换未不同的Class
                this.returnType = (Class<?>) resolvedReturnType;
            } else if (resolvedReturnType instanceof ParameterizedType) {
                this.returnType = (Class<?>) ((ParameterizedType) resolvedReturnType).getRawType();
            } else {
                this.returnType = method.getReturnType();
            }
            this.returnsVoid = void.class.equals(this.returnType);//是否返回空值
            this.returnsMany = configuration.getObjectFactory().isCollection(this.returnType) || this.returnType.isArray();//是否返回集合类型或者数组类型
            this.returnsCursor = Cursor.class.equals(this.returnType);//是否返回游标类型
            this.returnsOptional = Optional.class.equals(this.returnType);//是否返回Optional类型
            this.mapKey = getMapKey(method);//与mapKey注解有关
            this.returnsMap = this.mapKey != null;//是否返回Map集合
            this.rowBoundsIndex = getUniqueParamIndex(method, RowBounds.class);//获取分页参数在参数中的位置
            this.resultHandlerIndex = getUniqueParamIndex(method, ResultHandler.class);//获取ResultHandler在参数中的位置
            this.paramNameResolver = new ParamNameResolver(configuration, method);//初始化ParamNameResolver
        }
        ```

        - ###### ParamNameResolver的构造方法

          ```java
          public ParamNameResolver(Configuration config, Method method) {
              final Class<?>[] paramTypes = method.getParameterTypes();//获取参数列表的Class
              final Annotation[][] paramAnnotations = method.getParameterAnnotations();//获取参数上的注解，方便判断是否含有Param注解
              final SortedMap<Integer, String> map = new TreeMap<>();
              int paramCount = paramAnnotations.length;
              // get names from @Param annotations
              for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
                  if (isSpecialParameter(paramTypes[paramIndex])) {//判断该参数是否是特殊参数（如结果处理器、分页类）
                      // skip special parameters
                      continue;
                  }
                  String name = null;
                  for (Annotation annotation : paramAnnotations[paramIndex]) {
                      if (annotation instanceof Param) {//判断是否该参数上存在Param注解
                          hasParamAnnotation = true;
                          name = ((Param) annotation).value();
                          break;
                      }
                  }
                  if (name == null) {//没有Param注解的情况
                      // @Param was not specified.
                      if (config.isUseActualParamName()) {//查看configruation中是否指定采用方法签名上的原生参数名
                          name = getActualParamName(method, paramIndex);
                      }
                      if (name == null) {
                          // use the parameter index as the name ("0", "1", ...)
                          // gcode issue #71
                          name = String.valueOf(map.size());//如果configruation中指定不采用方法签名上的原生参数名则使用该参数在参数列表中的顺序编号作为参数名称
                      }
                  }
                  map.put(paramIndex, name);//放入Map集合中
              }
              names = Collections.unmodifiableSortedMap(map);//初始化Names
          }
          ```

  - ### MethodInvoker的invoke方法

    ```java
    public Object invoke(Object proxy, Method method, Object[] args, SqlSession sqlSession) throws Throwable {
        return mapperMethod.execute(sqlSession, args);//调用MethodInvoker内部的MapperMethod的execute方法
    }
    
    //execute方法
    public Object execute(SqlSession sqlSession, Object[] args) {
        Object result;
        switch (command.getType()) {
            case INSERT: {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = rowCountResult(sqlSession.insert(command.getName(), param));
                break;
            }
            case UPDATE: {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = rowCountResult(sqlSession.update(command.getName(), param));
                break;
            }
            case DELETE: {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = rowCountResult(sqlSession.delete(command.getName(), param));
                break;
            }
            case SELECT:
                if (method.returnsVoid() && method.hasResultHandler()) {
                    executeWithResultHandler(sqlSession, args);
                    result = null;
                } else if (method.returnsMany()) {
                    result = executeForMany(sqlSession, args);
                } else if (method.returnsMap()) {
                    result = executeForMap(sqlSession, args);
                } else if (method.returnsCursor()) {
                    result = executeForCursor(sqlSession, args);
                } else {
                    Object param = method.convertArgsToSqlCommandParam(args);
                    result = sqlSession.selectOne(command.getName(), param);
                    if (method.returnsOptional()
                        && (result == null || !method.getReturnType().equals(result.getClass()))) {
                        result = Optional.ofNullable(result);
                    }
                }
                break;
            case FLUSH:
                result = sqlSession.flushStatements();
                break;
            default:
                throw new BindingException("Unknown execution method for: " + command.getName());
        }
        if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
            throw new BindingException("Mapper method '" + command.getName()
                                       + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
        }
        return result;
    }
    ```

    - #### 调用MethodSignature的convertArgsToSqlCommandParam方法

      ```java
      public Object convertArgsToSqlCommandParam(Object[] args) {
          return paramNameResolver.getNamedParams(args);
      }
      
      public Object getNamedParams(Object[] args) {
          final int paramCount = names.size();//获取参数数量
          if (args == null || paramCount == 0) {//没有参数直接返回空
            return null;
          } else if (!hasParamAnnotation && paramCount == 1) {//一个参数直接返回第一个参数
            return args[names.firstKey()];
          } else {//多个参数则使用Map封装
            final Map<String, Object> param = new ParamMap<>();
            int i = 0;
            for (Map.Entry<Integer, String> entry : names.entrySet()) {
              param.put(entry.getValue(), args[entry.getKey()]);//根据之前参数解析时获取的参数名称和参数位置的Map构建参数的映射
              // 添加通用参数名称（param1，param2，...）
              final String genericParamName = GENERIC_NAME_PREFIX + (i + 1);
              // 确保不覆盖以@Param 命名的参数
              if (!names.containsValue(genericParamName)) {
                param.put(genericParamName, args[entry.getKey()]);
              }
              i++;
            }
            return param;
          }
        }
      ```

    - #### SqlSession的selectOne方法

      ```java
      public <T> T selectOne(String statement, Object parameter) {
          // Popular vote was to return null on 0 results and throw exception on too many.
          List<T> list = this.selectList(statement, parameter);
          if (list.size() == 1) {
              return list.get(0);
          } else if (list.size() > 1) {
              throw new TooManyResultsException("Expected one result (or null) to be returned by selectOne(), but found: " + list.size());
          } else {
              return null;
          }
      }
      
      //selectList方法
      public <E> List<E> selectList(String statement, Object parameter) {
          return this.selectList(statement, parameter, RowBounds.DEFAULT);
      }
      
      //重载selectList方法
      public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
          try {
              MappedStatement ms = configuration.getMappedStatement(statement);//从configuration中获取MappedStatement（同时会进行校验是否还有未解析的方法和Cache等）
              return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);//采用执行器执行MappedStatement
          } catch (Exception e) {
              throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
          } finally {
              ErrorContext.instance().reset();
          }
      }
      ```

      - ##### 执行器的query方法（如果有插件则会先进入插件的invoke方法）

        ```java
        public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
            BoundSql boundSql = ms.getBoundSql(parameterObject);
            CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);//创建一个CacheKey，其中包含了行界限，SQL语句，本次查询参数（对象封装），MappedStatement
            return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
        }
        ```

        - ###### MappedStatement的getBoundSql方法

          ```java
          public BoundSql getBoundSql(Object parameterObject) {
              BoundSql boundSql = sqlSource.getBoundSql(parameterObject);//根据成员变量获取BoundSql（这里是直接new了一个BoundSql）
              /*
              public StaticSqlSource(Configuration configuration, String sql, List<ParameterMapping> parameterMappings) {
              this.sql = sql;
              this.parameterMappings = parameterMappings;
              this.configuration = configuration;
            }
              */
              List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();//获取boundSql的参数Mappings
              if (parameterMappings == null || parameterMappings.isEmpty()) {//如果参数映射未空则直接新建一个boundSql（参数映射采用MappedStatement自身的参数映射）
                  boundSql = new BoundSql(configuration, boundSql.getSql(), parameterMap.getParameterMappings(), parameterObject);
              }
          
              //检查参数映射中的嵌套结果映射
              for (ParameterMapping pm : boundSql.getParameterMappings()) {
                  String rmId = pm.getResultMapId();
                  if (rmId != null) {
                      ResultMap rm = configuration.getResultMap(rmId);//参数映射中如果引用了ResultMap
                      if (rm != null) {
                          hasNestedResultMaps |= rm.hasNestedResultMaps();//是否含有嵌套的结果映射
                      }
                  }
              }
          
              return boundSql;
          }
          ```

        - ###### CachingExecutor的query方法

          ```java
          public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
              throws SQLException {
              Cache cache = ms.getCache();
              if (cache != null) {
                  flushCacheIfRequired(ms);//查看是否SQL语句上有指定需要清除缓存
                  if (ms.isUseCache() && resultHandler == null) {//查看SQL语句标签UseCache是否为true且ResultHandler为空
                      ensureNoOutParams(ms, boundSql);//确保没有输出类型的参数
                      @SuppressWarnings("unchecked")
                      List<E> list = (List<E>) tcm.getObject(cache, key);//在事务缓存管理器中通过Cachekey获取查询结果
                      if (list == null) {//如果查询结果未空则重新查询并放入Cache中
                          list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                          tcm.putObject(cache, key, list); // issue #578 and #116
                      }
                      return list;//返回查询结果
                  }
              }
              return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);//不采用Cache的查询
          }
          ```

          - query方法

            ```java
            public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
                ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
                if (closed) {//查看该执行器是否已关闭
                  throw new ExecutorException("Executor was closed.");
                }
                if (queryStack == 0 && ms.isFlushCacheRequired()) {//如果当前增加查询递归层数为0则清楚执行器的当地缓存
                  clearLocalCache();
                }
                List<E> list;
                try {
                  queryStack++;//增加查询递归层数
                  list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;//如果resultHandler为空则从本地缓存中获取查询结果
                  if (list != null) {
                    handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);//如果本地缓存中获取到的查询结果不为空则处理输出参数（输出参数仅限于存储过程）
                  } else {
                    list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);//本地缓存中结果为空则从数据库中查询
                  }
                } finally {
                  queryStack--;//查询完成减少查询层数
                }
                if (queryStack == 0) {//如果查询层数为0
                  for (DeferredLoad deferredLoad : deferredLoads) {//延迟加载将结果存储本地缓存中
                    deferredLoad.load();
                  }
                  // issue #601
                  deferredLoads.clear();//清楚延迟加载
                  if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {//如果本地缓存作用域是STATEMENT则清除缓存
                    // issue #482
                    clearLocalCache();
                  }
                }
                return list;
              }
            ```

            - queryFromDatabase方法
            
              ```java
              private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
                  List<E> list;
                  localCache.putObject(key, EXECUTION_PLACEHOLDER);//将EXECUTION_PLACEHOLDER放入本地缓存中
                  try {
                    list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
                  } finally {
                    localCache.removeObject(key);//移除缓存中的EXECUTION_PLACEHOLDER
                  }
                  localCache.putObject(key, list);//将查询结果放入缓存中
                  if (ms.getStatementType() == StatementType.CALLABLE) {//如果SQL类型是存储过程则同时放入localOutputParameterCache中
                    localOutputParameterCache.putObject(key, parameter);
                  }
                  return list;//返回查询结果
                }
              ```
            
              - doquery方法
            
                ```java
                public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
                    Statement stmt = null;
                    try {
                      Configuration configuration = ms.getConfiguration();
                      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);//新建StatementHandler类
                      stmt = prepareStatement(handler, ms.getStatementLog());//完成预编译和参数传入
                      return handler.query(stmt, resultHandler);//执行查询
                    } finally {
                      closeStatement(stmt);
                    }
                  }
                
                ```
                
                - configuration的newStatementHandler方法
                
                  ```java
                  public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
                      StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);//创建RoutingStatementHandler
                      statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);//插件加强
                      return statementHandler;//返回加强后的statementHandler
                  }
                  
                  //RoutingStatementHandler的构造方法
                  public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
                  
                      switch (ms.getStatementType()) {
                          case STATEMENT:
                              delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
                              break;
                          case PREPARED:
                              delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
                              break;
                          case CALLABLE:
                              delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
                              break;
                          default:
                              throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
                      }
                  
                  }
                  //simpleStatementHandler的构造方法
                  public SimpleStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
                      super(executor, mappedStatement, parameter, rowBounds, resultHandler, boundSql);
                  }
                  //BaseStatementHandler的构造方法
                  protected BaseStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
                      this.configuration = mappedStatement.getConfiguration();
                      this.executor = executor;
                      this.mappedStatement = mappedStatement;
                      this.rowBounds = rowBounds;
                  
                      this.typeHandlerRegistry = configuration.getTypeHandlerRegistry();
                      this.objectFactory = configuration.getObjectFactory();
                  
                      if (boundSql == null) { // issue #435, get the key before calculating the statement
                          generateKeys(parameterObject);//执行keyGenerator的方法（selectkey的使用），需要完善mappedStatement的boundSql
                          boundSql = mappedStatement.getBoundSql(parameterObject);
                      }
                  
                      this.boundSql = boundSql;
                  
                      this.parameterHandler = configuration.newParameterHandler(mappedStatement, parameterObject, boundSql);
                      this.resultSetHandler = configuration.newResultSetHandler(executor, mappedStatement, rowBounds, parameterHandler, resultHandler, boundSql);
                  }
                  ```
                
                - prepareStatement方法
                
                  ```java
                  private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {//statementLog是mybatis采用的日志类
                      Statement stmt;
                      Connection connection = getConnection(statementLog);//根据日志级别对Connection进行动态代理加强，以便打印日志
                      stmt = handler.prepare(connection, transaction.getTimeout());//预编译
                      handler.parameterize(stmt);//传入参数
                      return stmt;
                    }
                  ```
                
                  - StatementHandler的prepare方法
                
                    ```java
                    //RoutingStatementHandler的prepare方法
                    public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
                        return delegate.prepare(connection, transactionTimeout);
                    }
                    //BaseStatementHandler的prepare方法
                    public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
                        ErrorContext.instance().sql(boundSql.getSql());
                        Statement statement = null;
                        try {
                            statement = instantiateStatement(connection);//执行connection.prepareStatement方法
                            setStatementTimeout(statement, transactionTimeout);//设置超时时长
                            setFetchSize(statement);//设置语句结果集大小
                            return statement;//返回statement
                        } catch (SQLException e) {
                            closeStatement(statement);
                            throw e;
                        } catch (Exception e) {
                            closeStatement(statement);
                            throw new ExecutorException("Error preparing statement.  Cause: " + e, e);
                        }
                    }
                    ```
                
                    - PreparedStatementHandler的instantiateStatement方法
                
                      ```java
                      protected Statement instantiateStatement(Connection connection) throws SQLException {
                          String sql = boundSql.getSql();
                          if (mappedStatement.getKeyGenerator() instanceof Jdbc3KeyGenerator) {//如果含有KeyGenerator
                              String[] keyColumnNames = mappedStatement.getKeyColumns();//获取selelctkey中的KeyColumns
                              if (keyColumnNames == null) {
                                  return connection.prepareStatement(sql, PreparedStatement.RETURN_GENERATED_KEYS);
                              } else {
                                  return connection.prepareStatement(sql, keyColumnNames);
                              }
                          } else if (mappedStatement.getResultSetType() == ResultSetType.DEFAULT) {
                              return connection.prepareStatement(sql);//如果采用连接池则这里的Connection为代理对象会先进入PooledConnection的invoke方法
                          } else {
                              return connection.prepareStatement(sql, mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
                          }
                      }
                      ```
                
                      - Connection的prepareStatement方法(先进入了代理类的invoke方法)
                      
                        ```java
                        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                            String methodName = method.getName();
                            if (CLOSE.equals(methodName)) {
                              dataSource.pushConnection(this);//如果执行的是CLOSE方法则将连接从活跃池中移除并放入闲置池中
                              return null;
                            }
                            try {
                              if (!Object.class.equals(method.getDeclaringClass())) {//对ToString方法放行
                                // issue #579 toString() should never fail
                                // throw an SQLException instead of a Runtime
                                checkConnection();
                              }
                              return method.invoke(realConnection, args);//调用JDBC中的connection.prepareStatement方法
                            } catch (Throwable t) {
                              throw ExceptionUtil.unwrapThrowable(t);
                            }
                        
                          }
                        ```
                    
                  - parameterize方法
                  
                    ```java
                    public void parameterize(Statement statement) throws SQLException {
                        delegate.parameterize(statement);
                    }
                    public void parameterize(Statement statement) throws SQLException {
                        parameterHandler.setParameters((PreparedStatement) statement);
                    }
                    public void setParameters(PreparedStatement ps) {
                        ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
                        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();//获取参数列表
                        if (parameterMappings != null) {
                          for (int i = 0; i < parameterMappings.size(); i++) {
                            ParameterMapping parameterMapping = parameterMappings.get(i);
                            if (parameterMapping.getMode() != ParameterMode.OUT) {//如果参数类型不是输出类型
                              Object value;
                              String propertyName = parameterMapping.getProperty();//获取参数名称
                              if (boundSql.hasAdditionalParameter(propertyName)) { // 是否含有额外的参数
                                value = boundSql.getAdditionalParameter(propertyName);
                              } else if (parameterObject == null) {
                                value = null;
                              } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {//如果含有参数列表的Handler则赋值
                                value = parameterObject;
                              } else {
                                MetaObject metaObject = configuration.newMetaObject(parameterObject);//使用MetaObject包装参数类
                                value = metaObject.getValue(propertyName);//赋值
                              }
                              TypeHandler typeHandler = parameterMapping.getTypeHandler();//获取参数映射的处理类
                              JdbcType jdbcType = parameterMapping.getJdbcType();//获取JdbcType
                              if (value == null && jdbcType == null) {
                                jdbcType = configuration.getJdbcTypeForNull();//如果参数为空且jdbcType为空则从configuration中获取JDBC空类型并设置给jdbcType
                              }
                              try {
                                typeHandler.setParameter(ps, i + 1, value, jdbcType);//设置参数
                              } catch (TypeException | SQLException e) {
                                throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
                              }
                            }
                          }
                        }
                      }
                    ```
                  
                    - setParameter方法
                  
                      ```java
                      @Override
                      public void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
                          if (parameter == null) {
                              if (jdbcType == null) {
                                  throw new TypeException("JDBC requires that the JdbcType must be specified for all nullable parameters.");
                              }
                              try {
                                  ps.setNull(i, jdbcType.TYPE_CODE);//设置空类型
                              } catch (SQLException e) {
                                  throw new TypeException("Error setting null for parameter #" + i + " with JdbcType " + jdbcType + " . "
                                                          + "Try setting a different JdbcType for this parameter or a different jdbcTypeForNull configuration property. "
                                                          + "Cause: " + e, e);
                              }
                          } else {
                              try {
                                  setNonNullParameter(ps, i, parameter, jdbcType);//设置非空值
                              } catch (Exception e) {
                                  throw new TypeException("Error setting non null for parameter #" + i + " with JdbcType " + jdbcType + " . "
                                                          + "Try setting a different JdbcType for this parameter or a different configuration property. "
                                                          + "Cause: " + e, e);
                              }
                          }
                      }
                      ```
                  
                      - setNonNullParameter方法
                  
                        ```java
                        @Override
                        public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType)
                            throws SQLException {
                            ps.setString(i, parameter);//调用JDBC设置对应参数
                        }
                        ```
                  
                - query方法
                
                  ```java
                  
                  public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
                      PreparedStatement ps = (PreparedStatement) statement;
                      ps.execute();//执行JDBC
                      return resultSetHandler.handleResultSets(ps);//获取Mapper方法最终结果
                    }
                  ```
                
                  - handleResultSets方法
                
                    ```java
                    @Override
                      public List<Object> handleResultSets(Statement stmt) throws SQLException {
                        ErrorContext.instance().activity("handling results").object(mappedStatement.getId());
                    
                        final List<Object> multipleResults = new ArrayList<>();
                    
                        int resultSetCount = 0;//结果条数
                        ResultSetWrapper rsw = getFirstResultSet(stmt);//获取ResultSetWrapper
                    
                        List<ResultMap> resultMaps = mappedStatement.getResultMaps();//获取该mappedStatement需要的ResultMap数量
                        int resultMapCount = resultMaps.size();//返回的结果条数
                        validateResultMapsCount(rsw, resultMapCount);//校验ResultMap的数量至少为1（在指定了ResultMap的情况下）
                        while (rsw != null && resultMapCount > resultSetCount) {
                          ResultMap resultMap = resultMaps.get(resultSetCount);//获取ResultMap
                          handleResultSet(rsw, resultMap, multipleResults, null);//解析返回结果并将结果放入multipleResults中
                          rsw = getNextResultSet(stmt);
                          cleanUpAfterHandlingResultSet();
                          resultSetCount++;//已解析的ResultSet数量加一
                        }
                    
                        String[] resultSets = mappedStatement.getResultSets();//再次获取ResultSet（正常流程此时ResultSet已经关闭）
                        if (resultSets != null) {//嵌套查询才会出现此时resultSets不为空的情况
                          while (rsw != null && resultSetCount < resultSets.length) {
                            ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
                            if (parentMapping != null) {
                              String nestedResultMapId = parentMapping.getNestedResultMapId();
                              ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
                              handleResultSet(rsw, resultMap, null, parentMapping);
                            }
                            rsw = getNextResultSet(stmt);
                            cleanUpAfterHandlingResultSet();
                            resultSetCount++;
                          }
                        }
                    
                        return collapseSingleResultList(multipleResults);//如果返回结果的数量为1则直接返回multipleResults的第一个元素否则返回整个multipleResults
                      }
                    ```
                    
                    - getFirstResultSet方法
                    
                      ```java
                       private ResultSetWrapper getFirstResultSet(Statement stmt) throws SQLException {
                          ResultSet rs = stmt.getResultSet();//获取结果集
                          while (rs == null) {
                      
                            // 如果驱动程序没有将结果集作为第一个结果返回（HSQLDB 2.1），则继续获取第一个结果集
                            if (stmt.getMoreResults()) {
                              rs = stmt.getResultSet();
                            } else {
                              if (stmt.getUpdateCount() == -1) {
                                // 没有更多的结果。必须是没有结果集
                                break;
                              }
                            }
                          }
                          return rs != null ? new ResultSetWrapper(rs, configuration) : null;//如果结果集不为空则使用ResultSetWrapper包装结果集
                        }
                      ```
                    
                      - ResultSetWrapper类
                    
                        ```java
                        public class ResultSetWrapper {
                        
                          private final ResultSet resultSet;
                          private final TypeHandlerRegistry typeHandlerRegistry;
                          private final List<String> columnNames = new ArrayList<>();
                          private final List<String> classNames = new ArrayList<>();
                          private final List<JdbcType> jdbcTypes = new ArrayList<>();
                          private final Map<String, Map<Class<?>, TypeHandler<?>>> typeHandlerMap = new HashMap<>();
                          private final Map<String, List<String>> mappedColumnNamesMap = new HashMap<>();
                          private final Map<String, List<String>> unMappedColumnNamesMap = new HashMap<>();
                        
                          public ResultSetWrapper(ResultSet rs, Configuration configuration) throws SQLException {
                            super();
                            this.typeHandlerRegistry = configuration.getTypeHandlerRegistry();
                            this.resultSet = rs;
                            final ResultSetMetaData metaData = rs.getMetaData();//获取ResultSetMetaData（原生JDBC）
                            final int columnCount = metaData.getColumnCount();//获取列数
                            for (int i = 1; i <= columnCount; i++) {
                              columnNames.add(configuration.isUseColumnLabel() ? metaData.getColumnLabel(i) : metaData.getColumnName(i));//根据Configuration判断是否使用列标签
                              jdbcTypes.add(JdbcType.forCode(metaData.getColumnType(i)));//JDBCType枚举类中含有成员变量：Map<Integer,JdbcType> codeLookup = new HashMap<>();，用于表示JDBCType的Code
                              classNames.add(metaData.getColumnClassName(i));//获取结果集中结果的class名称
                            }
                          }
                        }
                        ```
                    
                    - handleResultSet方法
                    
                      ```java
                      private void handleResultSet(ResultSetWrapper rsw, ResultMap resultMap, List<Object> multipleResults, ResultMapping parentMapping) throws SQLException {
                          try {
                            if (parentMapping != null) {//有父类ResultMap的情况
                              handleRowValues(rsw, resultMap, null, RowBounds.DEFAULT, parentMapping);
                            } else {
                              if (resultHandler == null) {
                                DefaultResultHandler defaultResultHandler = new DefaultResultHandler(objectFactory);//创建DefaultResultHandler（其中使用了ObjectFactory创建了DefaultResultHandler内部成员变量list）
                                handleRowValues(rsw, resultMap, defaultResultHandler, rowBounds, null);//解析返回结果并放入ResultHandler的list中
                                multipleResults.add(defaultResultHandler.getResultList());//取出ResultHandler的list并添加到multipleResults中
                              } else {
                                handleRowValues(rsw, resultMap, resultHandler, rowBounds, null);
                              }
                            }
                          } finally {
                            // issue #228 (close resultsets)
                            closeResultSet(rsw.getResultSet());//关闭结果集
                          }
                        }
                      ```
                    
                      - handleRowValues方法
                    
                        ```java
                        public void handleRowValues(ResultSetWrapper rsw, ResultMap resultMap, ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping) throws SQLException {
                            if (resultMap.hasNestedResultMaps()) {//是否有嵌套的结果映射
                                ensureNoRowBounds();//检查分页是否合法
                                checkResultHandler();//确保ResultHandler不为空且没有指定ResultOrdered且开启了SafeResultHandler
                                handleRowValuesForNestedResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
                            } else {
                                handleRowValuesForSimpleResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);//将ResultSet中的内容转化为ResultMap指定的对象并存储在ResultHandler的list中
                            }
                        }
                        //handleRowValuesForSimpleResultMap方法
                        private void handleRowValuesForSimpleResultMap(ResultSetWrapper rsw, ResultMap resultMap, ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping)
                            throws SQLException {
                            DefaultResultContext<Object> resultContext = new DefaultResultContext<>();
                            ResultSet resultSet = rsw.getResultSet();
                            skipRows(resultSet, rowBounds);//根据分页情况跳过某些记录
                            while (shouldProcessMoreRows(resultContext, rowBounds) && !resultSet.isClosed() && resultSet.next()) {//shouldProcessMoreRows：确保记录条数小于指定条数
                                ResultMap discriminatedResultMap = resolveDiscriminatedResultMap(resultSet, resultMap, null);//从Configuration中获取鉴别器对象
                                Object rowValue = getRowValue(rsw, discriminatedResultMap, null);//将ResultSet包装为对象
                                storeObject(resultHandler, resultContext, rowValue, parentMapping, resultSet);//将该条返回结果存入resultContext和resultHandler中
                            }
                        }
                        ```
                        
                        - getRowValue方法
                        
                          ```java
                          private Object getRowValue(ResultSetWrapper rsw, ResultMap resultMap, String columnPrefix) throws SQLException {
                              final ResultLoaderMap lazyLoader = new ResultLoaderMap();
                              Object rowValue = createResultObject(rsw, resultMap, lazyLoader, columnPrefix);//创建ResultMap所指定的需要封装的对象（如果采用无参构造则是空模板，没有数据）
                              if (rowValue != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {//如果不含有能够处理resultMap指定封装对象的Handler
                                  final MetaObject metaObject = configuration.newMetaObject(rowValue);//使用MetaObject封装rowValue
                                  boolean foundValues = this.useConstructorMappings;//是否使用有参构造方法创建对象
                                  if (shouldApplyAutomaticMappings(resultMap, false)) {//是否采用了自动映射
                                      foundValues = applyAutomaticMappings(rsw, resultMap, metaObject, columnPrefix) || foundValues;
                                  }
                                  foundValues = applyPropertyMappings(rsw, resultMap, metaObject, lazyLoader, columnPrefix) || foundValues;
                                  foundValues = lazyLoader.size() > 0 || foundValues;
                                  rowValue = foundValues || configuration.isReturnInstanceForEmptyRow() ? rowValue : null;
                              }
                              return rowValue;
                          }
                          ```
                        
                          - createResultObject方法
                        
                            ```java
                            private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {
                                this.useConstructorMappings = false; // 重置之前的映射结果
                                final List<Class<?>> constructorArgTypes = new ArrayList<>();//创建构造方法参数类型list
                                final List<Object> constructorArgs = new ArrayList<>();//创建构造方法参数值list
                                Object resultObject = createResultObject(rsw, resultMap, constructorArgTypes, constructorArgs, columnPrefix);//创建结果对象（没有任何数据只是个空模板）
                                if (resultObject != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {//如果不含有结果对象类型的Handler
                                    final List<ResultMapping> propertyMappings = resultMap.getPropertyResultMappings();//获取ResultMap
                                    for (ResultMapping propertyMapping : propertyMappings) {
                                        // issue gcode #109 && issue #149
                                        if (propertyMapping.getNestedQueryId() != null && propertyMapping.isLazy()) {//检查ResultMap中是否存在懒加载如果有则通过configuration创建代理对象作为resultObject
                                            resultObject = configuration.getProxyFactory().createProxy(resultObject, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
                                            break;
                                        }
                                    }
                                }
                                this.useConstructorMappings = resultObject != null && !constructorArgTypes.isEmpty(); // set current mapping result
                                return resultObject;
                            }
                            
                            
                            //createResultObject方法
                            private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, List<Class<?>> constructorArgTypes, List<Object> constructorArgs, String columnPrefix)
                                throws SQLException {
                                final Class<?> resultType = resultMap.getType();//获取resultMap需要包装的对象类型
                                final MetaClass metaType = MetaClass.forClass(resultType, reflectorFactory);//resultType封装为MetaClass
                                final List<ResultMapping> constructorMappings = resultMap.getConstructorResultMappings();//获取构造方法Mapping
                                if (hasTypeHandlerForResultObject(rsw, resultType)) {//如果含有能够处理结果类型的Handler
                                    return createPrimitiveResultObject(rsw, resultMap, columnPrefix);//创建原始结果对象
                                } else if (!constructorMappings.isEmpty()) {//如果含有构造方法mapping
                                    return createParameterizedResultObject(rsw, resultType, constructorMappings, constructorArgTypes, constructorArgs, columnPrefix);
                                } else if (resultType.isInterface() || metaType.hasDefaultConstructor()) {//如果resultType是接口并且含有默认的构造方法
                                    return objectFactory.create(resultType);//采用objectFactory创建
                                } else if (shouldApplyAutomaticMappings(resultMap, false)) {
                                    return createByConstructorSignature(rsw, resultType, constructorArgTypes, constructorArgs);
                                }
                                throw new ExecutorException("Do not know how to create an instance of " + resultType);
                            }
                            ```
                        
                          - applyAutomaticMappings方法
                        
                            ```java
                            private boolean applyAutomaticMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject, String columnPrefix) throws SQLException {
                                List<UnMappedColumnAutoMapping> autoMapping = createAutomaticMappings(rsw, resultMap, metaObject, columnPrefix);//创建一个空的自动映射Mapping
                                boolean foundValues = false;
                                if (!autoMapping.isEmpty()) {
                                    for (UnMappedColumnAutoMapping mapping : autoMapping) {
                                        final Object value = mapping.typeHandler.getResult(rsw.getResultSet(), mapping.column);//获取该列的值
                                        if (value != null) {
                                            foundValues = true;
                                        }
                                        if (value != null || (configuration.isCallSettersOnNulls() && !mapping.primitive)) {
                                            // gcode 问题 377，在 null 上调用 setter（值不是“找到”）
                                            metaObject.setValue(mapping.property, value);//设置值
                                        }
                                    }
                                }
                                return foundValues;
                            }
                            ```
                        
                            - createAutomaticMappings方法
                        
                              ```java
                              private List<UnMappedColumnAutoMapping> createAutomaticMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject, String columnPrefix) throws SQLException {
                                  final String mapKey = resultMap.getId() + ":" + columnPrefix;//拼接Map的key：resultMap的ID+列前缀
                                  List<UnMappedColumnAutoMapping> autoMapping = autoMappingsCache.get(mapKey);//查看缓存中是否存在该mapkey对应的自动映射Map
                                  if (autoMapping == null) {//缓存中没有的情况下
                                      autoMapping = new ArrayList<>();//用于存储未映射列名的集合
                                      final List<String> unmappedColumnNames = rsw.getUnmappedColumnNames(resultMap, columnPrefix);//获取未映射的列名集合（通过MapKey）
                                      for (String columnName : unmappedColumnNames) {//对于未映射列名的集合的处理
                                          String propertyName = columnName;
                                          if (columnPrefix != null && !columnPrefix.isEmpty()) {//含有列名前缀的前提下
                                              //
                                              // 指定 columnPrefix 时，忽略不带前缀的列
                                              if (columnName.toUpperCase(Locale.ENGLISH).startsWith(columnPrefix)) {//如果作为列前缀开头
                                                  propertyName = columnName.substring(columnPrefix.length());//去除列前缀
                                              } else {
                                                  continue;
                                              }
                                          }
                                          final String property = metaObject.findProperty(propertyName, configuration.isMapUnderscoreToCamelCase());//从resultMap找到该值
                                          if (property != null && metaObject.hasSetter(property)) {//如果在metaObject中找到了该成员变量并且该成员变量含有set方法则
                                              if (resultMap.getMappedProperties().contains(property)) {//排除mappedProperties中的属性
                                                  continue;
                                              }
                                              final Class<?> propertyType = metaObject.getSetterType(property);//获取set方法参数的类型
                                              if (typeHandlerRegistry.hasTypeHandler(propertyType, rsw.getJdbcType(columnName))) {//查看是否含有set方法参数类型的TypeHandler
                                                  final TypeHandler<?> typeHandler = rsw.getTypeHandler(propertyType, columnName);
                                                  autoMapping.add(new UnMappedColumnAutoMapping(columnName, property, typeHandler, propertyType.isPrimitive()));//向自动结果映射中添加该记录
                                              } else {
                                                  //AutoMappingUnknownColumnBehavior指的是当自动映射未能找到对应的列名时采用的行为
                                                  //一共三种行为NONE: 不做任何反应
                              //WARNING: 输出警告日志（'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior' 的日志等级必须设置为 WARN）
                              //FAILING: 映射失败 (抛出 SqlSessionException)
                                                  configuration.getAutoMappingUnknownColumnBehavior()
                                                      .doAction(mappedStatement, columnName, property, propertyType);//记录日志
                                              }
                                          } else {
                                              configuration.getAutoMappingUnknownColumnBehavior()
                                                  .doAction(mappedStatement, columnName, (property != null) ? property : propertyName, null);
                                          }
                                      }
                                      autoMappingsCache.put(mapKey, autoMapping);//将该自动映射Mapping放入缓存中
                                  }
                                  return autoMapping;
                              }
                              ```
                        
                          - applyPropertyMappings方法
                        
                            ```java
                            private boolean applyPropertyMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject, ResultLoaderMap lazyLoader, String columnPrefix)
                                throws SQLException {
                                final List<String> mappedColumnNames = rsw.getMappedColumnNames(resultMap, columnPrefix);//获取ResultSet中拼接后的大写的列名
                                boolean foundValues = false;
                                final List<ResultMapping> propertyMappings = resultMap.getPropertyResultMappings();//获取映射集合
                                for (ResultMapping propertyMapping : propertyMappings) {
                                    String column = prependPrefix(propertyMapping.getColumn(), columnPrefix);//拼接列名（ResultMap中）
                                    if (propertyMapping.getNestedResultMapId() != null) {
                                        // the user added a column attribute to a nested result map, ignore it
                                        column = null;
                                    }
                                    if (propertyMapping.isCompositeResult()//如果是复合结果（一个ResultMapping中含有多个ResultMapping）
                                        || (column != null && mappedColumnNames.contains(column.toUpperCase(Locale.ENGLISH)))//列名不为空且ResultSet中含有ResultMap的列名
                                        || propertyMapping.getResultSet() != null) {//ResultMap中还没有存储ResultSet
                                        Object value = getPropertyMappingValue(rsw.getResultSet(), metaObject, propertyMapping, lazyLoader, columnPrefix);//获取该列的值
                                        // issue #541 make property optional
                                        final String property = propertyMapping.getProperty();//获取结果类中该成员的名称
                                        if (property == null) {
                                            continue;
                                        } else if (value == DEFERRED) {
                                            foundValues = true;
                                            continue;
                                        }
                                        if (value != null) {
                                            foundValues = true;
                                        }
                                        if (value != null || (configuration.isCallSettersOnNulls() && !metaObject.getSetterType(property).isPrimitive())) {
                                            // gcode issue #377, call setter on nulls (value is not 'found')
                                            metaObject.setValue(property, value);//执行set方法注入
                                        }
                                    }
                                }
                                return foundValues;
                            }
                            ```
                        
                        - storeObject方法
                        
                          ```java
                          private void storeObject(ResultHandler<?> resultHandler, DefaultResultContext<Object> resultContext, Object rowValue, ResultMapping parentMapping, ResultSet rs) throws SQLException {
                              if (parentMapping != null) {//如果含有父类ResultMap
                                  linkToParents(rs, parentMapping, rowValue);
                              } else {
                                  callResultHandler(resultHandler, resultContext, rowValue);//将结果存放到resultContext和resultHandler中
                              }
                          }
                          //callResultHandler方法
                          private void callResultHandler(ResultHandler<?> resultHandler, DefaultResultContext<Object> resultContext, Object rowValue) {
                              resultContext.nextResultObject(rowValue);//resultContext中存储返回结果和返回结果条数，这一步是增加返回结果条数并为返回结果（resultContext中的Object）赋值
                              ((ResultHandler<Object>) resultHandler).handleResult(resultContext);//resultHandler中含有一个List<Object>用于存储所有返回结果，一条结果一个Obj，多条结果用List存储
                          }
                          ```
                        
                          
                    
                    
                    
                    