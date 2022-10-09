# Mybatis（5）延迟加载

- ## 主配置文件

  ```xml
  <settings>
          <setting name="logImpl" value="STDOUT_LOGGING"/>
          <setting name="lazyLoadingEnabled" value="true"/>
          <setting name="aggressiveLazyLoading" value="false"/>
      <!--name:lazyLoadTriggerMethods	指定对象的哪些方法触发一次延迟加载。	value：用逗号分隔的方法列表。	默认值equals,clone,hashCode,toString-->
      <setting name="lazyLoadTriggerMethods" value="toString"/>
      
      </settings>
  ```

  - 除了主配置文件之外还可以在ResultMap中单独指定是否延迟加载

    ```xml
    <!--对于含有嵌套查询（select）的语句可以指定fetchType属性（lazy/eager），该选项可以覆盖configuration中的（也即是settings）中的懒加载配置（指的是如果configuration中没有指定开启懒加载且fetchType中采用懒加载则对于该次查询会开启懒加载）
    -->
    <association property="user" column="u_id" javaType="User" select="generator.mapper.UserMapper.selectByPrimaryKey" fetchType="lazy"/>
    ```

    - 源码片段

      ```java
      //在XMLMapperBuilder中的buildResultMappingFromContext方法中
      //在 resultMapElements(context.evalNodes("/mapper/resultMap"));该语句中调用过该方法（buildResultMappingFromContext），也就是在解析XML文件中的ResultMap属性时
      boolean lazy = "lazy".equals(context.getStringAttribute("fetchType", configuration.isLazyLoadingEnabled() ? "lazy" : "eager"));
      
      //在ResultMapping类中的Builder方法
      resultMapping.lazy = configuration.isLazyLoadingEnabled();
      ```

- ## 延迟加载的执行原理

  - （1）parseConfiguration方法
  
    ```java
    settingsElement(settings);//读取Setting标签中的延迟加载值
    mapperElement(root.evalNode("mappers"));//设置ResultMap的延迟记载
    ```
  
  - （2）configuration初始化
  
    ```java
    configuration.setLazyLoadingEnabled(booleanValueOf(props.getProperty("lazyLoadingEnabled"), false));
    configuration.setAggressiveLazyLoading(booleanValueOf(props.getProperty("aggressiveLazyLoading"), false));
    configuration.setLazyLoadTriggerMethods(stringSetValueOf(props.getProperty("lazyLoadTriggerMethods"), "equals,clone,hashCode,toString"));
    ```
  
  - (3)mapperElement读取Mapper文件
  
    ```java
    private void mapperElement(XNode parent) throws Exception {
        if (parent != null) {
          for (XNode child : parent.getChildren()) {
            if ("package".equals(child.getName())) {
              String mapperPackage = child.getStringAttribute("name");
              configuration.addMappers(mapperPackage);
            } else {
              String resource = child.getStringAttribute("resource");
              String url = child.getStringAttribute("url");
              String mapperClass = child.getStringAttribute("class");
              if (resource != null && url == null && mapperClass == null) {
                ErrorContext.instance().resource(resource);
                InputStream inputStream = Resources.getResourceAsStream(resource);
                XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
                mapperParser.parse();//Mapper.xml文件解析
              } else if (resource == null && url != null && mapperClass == null) {
                ErrorContext.instance().resource(url);
                InputStream inputStream = Resources.getUrlAsStream(url);
                XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
                mapperParser.parse();
              } else if (resource == null && url == null && mapperClass != null) {
                Class<?> mapperInterface = Resources.classForName(mapperClass);
                configuration.addMapper(mapperInterface);
              } else {
                throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
              }
            }
          }
        }
      }
    ```
  
  - （4）parse方法
  
    ```java
    public void parse() {
        if (!configuration.isResourceLoaded(resource)) {
            configurationElement(parser.evalNode("/mapper"));
            configuration.addLoadedResource(resource);
            bindMapperForNamespace();
        }
    
        parsePendingResultMaps();
        parsePendingCacheRefs();
        parsePendingStatements();
    }
    ```
  
  - (5)configurationElement方法
  
    ```java
    private void configurationElement(XNode context) {
        try {
            String namespace = context.getStringAttribute("namespace");
            if (namespace == null || namespace.equals("")) {
                throw new BuilderException("Mapper's namespace cannot be empty");
            }
            builderAssistant.setCurrentNamespace(namespace);
            cacheRefElement(context.evalNode("cache-ref"));
            cacheElement(context.evalNode("cache"));
            parameterMapElement(context.evalNodes("/mapper/parameterMap"));
            resultMapElements(context.evalNodes("/mapper/resultMap"));//解析ResultMap
            sqlElement(context.evalNodes("/mapper/sql"));
            buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
        } catch (Exception e) {
            throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
        }
    }
    ```
  
  - （6）resultMapElements方法
  
    ```java
    private ResultMap resultMapElement(XNode resultMapNode, List<ResultMapping> additionalResultMappings, Class<?> enclosingType) {
        ErrorContext.instance().activity("processing " + resultMapNode.getValueBasedIdentifier());
        String type = resultMapNode.getStringAttribute("type",
                                                       resultMapNode.getStringAttribute("ofType",
                                                                                        resultMapNode.getStringAttribute("resultType",
                                                                                                                         resultMapNode.getStringAttribute("javaType"))));
        Class<?> typeClass = resolveClass(type);
        if (typeClass == null) {
            typeClass = inheritEnclosingType(resultMapNode, enclosingType);//看不懂
        }
        Discriminator discriminator = null;//鉴别器
        List<ResultMapping> resultMappings = new ArrayList<>(additionalResultMappings);
        List<XNode> resultChildren = resultMapNode.getChildren();
        for (XNode resultChild : resultChildren) {//遍历ResultMap下的所有子标签
            if ("constructor".equals(resultChild.getName())) {//解析构造标签
                processConstructorElement(resultChild, typeClass, resultMappings);
            } else if ("discriminator".equals(resultChild.getName())) {//解析鉴别器标签
                discriminator = processDiscriminatorElement(resultChild, typeClass, resultMappings);
            } else {//解析普通标签
                List<ResultFlag> flags = new ArrayList<>();
                if ("id".equals(resultChild.getName())) {
                    flags.add(ResultFlag.ID);
                }
                resultMappings.add(buildResultMappingFromContext(resultChild, typeClass, flags));//向resultMappings中添加该标签
            }
        }
        String id = resultMapNode.getStringAttribute("id",
                                                     resultMapNode.getValueBasedIdentifier());
        String extend = resultMapNode.getStringAttribute("extends");
        Boolean autoMapping = resultMapNode.getBooleanAttribute("autoMapping");
        ResultMapResolver resultMapResolver = new ResultMapResolver(builderAssistant, id, typeClass, extend, discriminator, resultMappings, autoMapping);
        try {
            return resultMapResolver.resolve();
        } catch (IncompleteElementException  e) {
            configuration.addIncompleteResultMap(resultMapResolver);
            throw e;
        }
    }
    ```
  
  - （7）buildResultMappingFromContext方法
  
    ```java
    private ResultMapping buildResultMappingFromContext(XNode context, Class<?> resultType, List<ResultFlag> flags) {
    //省略
        String nestedSelect = context.getStringAttribute("select");//嵌套Select
        boolean lazy = "lazy".equals(context.getStringAttribute("fetchType", configuration.isLazyLoadingEnabled() ? "lazy" : "eager"));//对于ResultMap中可以通过fetchType的设置指定某个关联查询（select属性）进行懒加载，且该属性可以覆盖Configuration中的懒加载属性
        //省略
        return builderAssistant.buildResultMapping(resultType, property, column, javaTypeClass, jdbcTypeEnum, nestedSelect, nestedResultMap, notNullColumn, columnPrefix, typeHandlerClass, flags, resultSet, foreignColumn, lazy);
    }
    ```
  
  - （8）buildResultMapping方法
  
    ```java
    public ResultMapping buildResultMapping(
        Class<?> resultType,
        String property,
        String column,
        Class<?> javaType,
        JdbcType jdbcType,
        String nestedSelect,
        String nestedResultMap,
        String notNullColumn,
        String columnPrefix,
        Class<? extends TypeHandler<?>> typeHandler,
        List<ResultFlag> flags,
        String resultSet,
        String foreignColumn,
        boolean lazy) {
        Class<?> javaTypeClass = resolveResultJavaType(resultType, property, javaType);
        TypeHandler<?> typeHandlerInstance = resolveTypeHandler(javaTypeClass, typeHandler);
        List<ResultMapping> composites;
        if ((nestedSelect == null || nestedSelect.isEmpty()) && (foreignColumn == null || foreignColumn.isEmpty())) {
            composites = Collections.emptyList();//如果没有select和foreignColumn则直接将composites设置为空
        } else {
            composites = parseCompositeColumnName(column);//如果含有嵌套查询则进入
        }
        return new ResultMapping.Builder(configuration, property, column, javaTypeClass)
            .jdbcType(jdbcType)
            .nestedQueryId(applyCurrentNamespace(nestedSelect, true))
            .nestedResultMapId(applyCurrentNamespace(nestedResultMap, true))
            .resultSet(resultSet)
            .typeHandler(typeHandlerInstance)
            .flags(flags == null ? new ArrayList<>() : flags)
            .composites(composites)
            .notNullColumns(parseMultipleColumnNames(notNullColumn))
            .columnPrefix(columnPrefix)
            .foreignColumn(foreignColumn)
            .lazy(lazy)
            .build();
    }
    ```
  
  - （9）parseCompositeColumnName方法
  
    ```java
    private List<ResultMapping> parseCompositeColumnName(String columnName) {//columnName是select属性所在标签上指定的columnName
        List<ResultMapping> composites = new ArrayList<>();
        if (columnName != null && (columnName.indexOf('=') > -1 || columnName.indexOf(',') > -1)) {
            StringTokenizer parser = new StringTokenizer(columnName, "{}=, ", false);
            while (parser.hasMoreTokens()) {
                String property = parser.nextToken();
                String column = parser.nextToken();
                ResultMapping complexResultMapping = new ResultMapping.Builder(
                    configuration, property, column, configuration.getTypeHandlerRegistry().getUnknownTypeHandler()).build();
                composites.add(complexResultMapping);
            }
        }
        return composites;
    }
    ```
  
    