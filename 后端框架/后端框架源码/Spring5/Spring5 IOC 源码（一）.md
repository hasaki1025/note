# Spring5源码分析（一）

- 入口

  ```java
  public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
      this();
      register(componentClasses);
      refresh();
  }
  ```

  - 构造方法（AnnotationConfigApplicationContext）

    - AnnotatedBeanDefinitionReader的创建

      ```java
      public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
          this(registry, getOrCreateEnvironment(registry));//AnnotationConfigApplicationContext继承了BeanDefinitionRegistry，将该ApplicationContext作为BeanDefinitionRegistry
      }
      
      public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
          Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
          Assert.notNull(environment, "Environment must not be null");
          this.registry = registry;
          this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);//通过registry和环境创建Conditional注解评估器
          AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);//注册8个注解
      }
      ```

    - 注册传入的组件类Class

      ```java
      register(componentClasses);
      
      public void register(Class<?>... componentClasses) {
          Assert.notEmpty(componentClasses, "At least one component class must be specified");
          StartupStep registerComponentClass = this.getApplicationStartup().start("spring.context.component-classes.register")
              .tag("classes", () -> Arrays.toString(componentClasses));
          this.reader.register(componentClasses);
          registerComponentClass.end();
      }
      ```

      - 使用AnnotatedBeanDefinitionReader注册传入的组件类

        ```java
        public void register(Class<?>... componentClasses) {
            for (Class<?> componentClass : componentClasses) {
                registerBean(componentClass);
            }
        }
        ```

        - registerBean方法

          ```java
          	private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
          			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
          			@Nullable BeanDefinitionCustomizer[] customizers) {
          
          		AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);//创建beanClass的BeanDefinition
          		if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {//判断该Bean是否可以不加载，如果beanClass上有Conditional注解则不能跳过
          			return;
          		}
          
          		abd.setInstanceSupplier(supplier);//设置Bean回调
          		ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);//解析Bean的作用域
          		abd.setScope(scopeMetadata.getScopeName());//设置作用域
          		String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
          
          		AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);//检查特殊元注解
          		if (qualifiers != null) {//如果参数上含有指定的其他注解
          			for (Class<? extends Annotation> qualifier : qualifiers) {
          				if (Primary.class == qualifier) {//参数中是否含有Primary注解
          					abd.setPrimary(true);
          				}
          				else if (Lazy.class == qualifier) {//参数中是否含有lazy注解
          					abd.setLazyInit(true);
          				}
          				else {
          					abd.addQualifier(new AutowireCandidateQualifier(qualifier));//该Bean在qualifier注解上的条件添加
          				}
          			}
          		}
          		if (customizers != null) {//用于定制工厂的BeanDefinition的一个或多个回调，例如设置惰性初始化或主标志
          			for (BeanDefinitionCustomizer customizer : customizers) {
          				customizer.customize(abd);
          			}
          		}
          
          		BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);//创建BeanDefinitionHolder包装BeanDefinitionHoldere
          		definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);//应用范围代理模式（根据代理方式返回代理后的对象，如果不采用代理不修改）
          		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);//注册该BeanDefinition
          	}
          ```

          - registerBeanDefinition方法

            ```java
            public static void registerBeanDefinition(
                BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
                throws BeanDefinitionStoreException {
            
                // 在主名称下注册 bean 定义。
                String beanName = definitionHolder.getBeanName();
                registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());//在Bean工厂中注册该BeanDefinition
            
                //注册 bean 名称的别名（如果有）
                String[] aliases = definitionHolder.getAliases();
                if (aliases != null) {
                    for (String alias : aliases) {
                        registry.registerAlias(beanName, alias);
                    }
                }
            }
            
            public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
                throws BeanDefinitionStoreException {
            
                Assert.hasText(beanName, "Bean name must not be empty");
                Assert.notNull(beanDefinition, "BeanDefinition must not be null");
            
                if (beanDefinition instanceof AbstractBeanDefinition) {
                    try {
                        ((AbstractBeanDefinition) beanDefinition).validate();//校验重载方法
                    }
                    catch (BeanDefinitionValidationException ex) {
                        throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
                                                               "Validation of bean definition failed", ex);
                    }
                }
            
                BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
                if (existingDefinition != null) {//重复Bean定义解决
                    if (!isAllowBeanDefinitionOverriding()) {//是否允许重新注册具有相同名称的不同定义。
                        throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
                    }
                    else if (existingDefinition.getRole() < beanDefinition.getRole()) {//在IOC容器中如果允许重新注册具有相同的名称的不同定义的Bean，则比较从beanDefinitionMap获取的BeanDefinition和传入的BeanDefinition的角色重要性，优先级高的覆盖优先级低的
                        // 例如是 ROLE_APPLICATION，现在被 ROLE_SUPPORT 或 ROLE_INFRASTRUCTURE 覆盖
                        if (logger.isInfoEnabled()) {
                            logger.info("Overriding user-defined bean definition for bean '" + beanName +
                                        "' with a framework-generated bean definition: replacing [" +
                                        existingDefinition + "] with [" + beanDefinition + "]");
                        }
                    }
                    else if (!beanDefinition.equals(existingDefinition)) {//如果两个beanDefinition不同则无需考虑覆盖问题
                        if (logger.isDebugEnabled()) {
                            logger.debug("Overriding bean definition for bean '" + beanName +
                                         "' with a different definition: replacing [" + existingDefinition +
                                         "] with [" + beanDefinition + "]");
                        }
                    }
                    else {//如果重要性相同但是两者不一样则打印trace日志（等级低于Debug）
                        if (logger.isTraceEnabled()) {
                            logger.trace("Overriding bean definition for bean '" + beanName +
                                         "' with an equivalent definition: replacing [" + existingDefinition +
                                         "] with [" + beanDefinition + "]");
                        }
                    }
                    this.beanDefinitionMap.put(beanName, beanDefinition);//放入beanDefinitionMap中
                }
                else {//如果Map中不含有该名称的Bean定义则放入Map中
                    if (hasBeanCreationStarted()) {//如果在Bean创建阶段
                        // 无法再修改启动时集合元素（用于稳定迭代）
                        synchronized (this.beanDefinitionMap) {
                            this.beanDefinitionMap.put(beanName, beanDefinition);
                            List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
                            updatedDefinitions.addAll(this.beanDefinitionNames);
                            updatedDefinitions.add(beanName);
                            this.beanDefinitionNames = updatedDefinitions;//为什么不直接添加？
                            removeManualSingletonName(beanName);//删除工厂的内部手动单例名称集。//TODO 暂时不知道有什么作用
                        }
                    }
                    else {
                        // 仍处于启动注册阶段
                        this.beanDefinitionMap.put(beanName, beanDefinition);
                        this.beanDefinitionNames.add(beanName);
                        removeManualSingletonName(beanName);//删除手动单例名称 //TODO 暂时不知道有什么作用
                    }
                    this.frozenBeanDefinitionNames = null;//frozenBeanDefinitionNames：在冻结配置的情况下缓存 bean 定义名称的数组（不知道有什么作用）
                }
            
                if (existingDefinition != null || containsSingleton(beanName)) {//如果存在单例或者之前含有注册过的BeanDefiniton则重新设置Bean的定义
                    resetBeanDefinition(beanName);//移除之前的Bean实例和BeanDefinition
                }
                else if (isConfigurationFrozen()) {
                    clearByTypeCache();
                }
            }
            ```

            

