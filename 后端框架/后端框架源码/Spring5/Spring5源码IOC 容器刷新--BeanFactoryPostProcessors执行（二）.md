# Spring5源码IOC 容器刷新--BeanFactoryPostProcessors执行（二）

- 入口

  ```java
  invokeBeanFactoryPostProcessors(beanFactory);
  ```

  - 执行所有后置处理器方法

    ```java
    protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
        PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());//调用 Bean Factory 后处理器
    
        //检测 LoadTimeWeaver 并准备编织（如果同时发现）
        //（例如，通过 ConfigurationClassPostProcessor 注册的 @Bean 方法）
        if (!NativeDetector.inNativeImage() && beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
            beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
            beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
        }
    }
    ```

    - invokeBeanFactoryPostProcessors方法

      ```java
      public static void invokeBeanFactoryPostProcessors(
      			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
      
      		Set<String> processedBeans = new HashSet<>();//用于保存已执行的后置处理器器的Bean名称
      
      		if (beanFactory instanceof BeanDefinitionRegistry) {
      			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
      			List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();//常规后处理器
      			List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();//注册表处理器（BeanFactoryPostProcessor的子类）
      
      			for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {//遍历所有参数中传递的BeanFactoryPostProcessor（一些内部的后置处理器，本次执行中没有）
      				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {//如果该处理器是BeanDefinition注册器后置处理器的实现类则强制转换后放入registryProcessors中
      					BeanDefinitionRegistryPostProcessor registryProcessor =
      							(BeanDefinitionRegistryPostProcessor) postProcessor;
      					registryProcessor.postProcessBeanDefinitionRegistry(registry);// 调用后置处理方法（BeanDefinitionRegistryPostProcessor类型的）
      					registryProcessors.add(registryProcessor);
      				}
      				else {
      					regularPostProcessors.add(postProcessor);//否则就直接加入regularPostProcessors中
      				}
      			}
      
      			// Do not initialize FactoryBeans here: We need to leave all regular beans
      			// uninitialized to let the bean factory post-processors apply to them!
      			// Separate between BeanDefinitionRegistryPostProcessors that implement
      			// PriorityOrdered, Ordered, and the rest.不要在此处初始化 FactoryBeans：我们需要让所有常规 bean 保持未初始化状态，以便 bean 工厂后处理器应用到它们！将实现 PriorityOrdered、Ordered 和其余部分的 BeanDefinitionRegistryPostProcessor 分开。
      			List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();//当前需要执行的注册表处理器
      
      			// 首先，调用实现 PriorityOrdered 的 BeanDefinitionRegistryPostProcessor
      			String[] postProcessorNames =
      					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);//从BeanFactory中获取注册表处理器
      			for (String ppName : postProcessorNames) {//遍历BeanFactory中的注册表处理器
      				if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {//该Bean是否需要排序（根据PriorityOrdered设置注入顺序）
      					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));//向currentRegistryProcessors添加该Bean
      					processedBeans.add(ppName);//添加到processedBeans
      				}
      			}
      			sortPostProcessors(currentRegistryProcessors, beanFactory);//排序
      			registryProcessors.addAll(currentRegistryProcessors);
      			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());//执行所有注册表后置处理器
      			currentRegistryProcessors.clear();//清空
      
      			// 接下来，调用实现 Ordered 的 BeanDefinitionRegistryPostProcessors。
      			postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);//再次获取BeanDefinitionRegistryPostProcessor的所有Bean名称
      			for (String ppName : postProcessorNames) {
      				if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {//实现了Odered的BeanDefinitionRegistryPostProcessors被放入currentRegistryProcessors中
      					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
      					processedBeans.add(ppName);//添加到已执行的集合中
      				}
      			}
      			sortPostProcessors(currentRegistryProcessors, beanFactory);
      			registryProcessors.addAll(currentRegistryProcessors);
      			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());//调用后置处理方法
      			currentRegistryProcessors.clear();//清空
      
      			//最后，调用所有其他 BeanDefinitionRegistryPostProcessors 直到不再出现。
      			boolean reiterate = true;
      			while (reiterate) {
      				reiterate = false;
      				postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
      				for (String ppName : postProcessorNames) {
      					if (!processedBeans.contains(ppName)) {//检查是否已经调用过
      						currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));//添加至当前需要执行的后置处理器的集合中
      						processedBeans.add(ppName);//添加已执行
      						reiterate = true;//如果出现了新的注册后置处理器则需要继续循环
      					}
      				}
      				sortPostProcessors(currentRegistryProcessors, beanFactory);
      				registryProcessors.addAll(currentRegistryProcessors);
      				invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
      				currentRegistryProcessors.clear();
      			}
      
      			// 现在，调用到目前为止处理的所有处理器的 postProcessBeanFactory 回调。
      			invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
      			invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
      		}
      
      		else {//如果该BeanFactory不是Bean定义注册器
      			// 调用向上下文实例注册的工厂处理器。
      			invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
      		}
      
      		//调用向上下文实例注册的工厂处理器。
      		// 未初始化让bean factory后处理器应用到他们身上！
      		String[] postProcessorNames =//获取BeanFactory中所有BeanFactoryPostProcessor(普通的)
      				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);
      
      		// Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
      		// Ordered, and the rest.将实现 PriorityOrdered、Ordered 和其余部分的 BeanFactoryPostProcessor 分开。
      		List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
      		List<String> orderedPostProcessorNames = new ArrayList<>();
      		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
      		for (String ppName : postProcessorNames) {//遍历所有后置处理器并将其分开
      			if (processedBeans.contains(ppName)) {//保证不重复执行
      				// 跳过 - 已在上述第一阶段处理
      			}
      			else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {//实现了PriorityOrdered的分一组
      				priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
      			}
      			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {//实现了Ordered的分一组
      				orderedPostProcessorNames.add(ppName);
      			}
      			else {
      				nonOrderedPostProcessorNames.add(ppName);//什么都没有的分一组
      			}
      		}
      
      		// 首先，调用实现 PriorityOrdered 的 BeanFactoryPostProcessor。
      		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
      		invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);
      
      		// 接下来，调用实现 Ordered 的 BeanFactoryPostProcessors。
      		List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
      		for (String postProcessorName : orderedPostProcessorNames) {
      			orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
      		}
      		sortPostProcessors(orderedPostProcessors, beanFactory);
      		invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);
      
      		// 最后，调用所有其他 BeanFactoryPostProcessor。
      		List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
      		for (String postProcessorName : nonOrderedPostProcessorNames) {
      			nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
      		}
      		invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);
      
      		// 清除缓存的合并 bean 定义，因为后处理器可能有
      		//修改了原始元数据，例如替换值中的占位符..
      		beanFactory.clearMetadataCache();
      	}
      ```

      - 在本次案例中BeanDefinitionRegistryPostProcessor（且实现了PriorityOrdered）只有一个，也就是与Configuration注解相关的后置处理器

        ```java
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());//执行所有注册表后置处理器
        
        private static void invokeBeanDefinitionRegistryPostProcessors(
            Collection<? extends BeanDefinitionRegistryPostProcessor> postProcessors, BeanDefinitionRegistry registry, ApplicationStartup applicationStartup) {
        
            for (BeanDefinitionRegistryPostProcessor postProcessor : postProcessors) {//执行所有后置处理方法
                StartupStep postProcessBeanDefRegistry = applicationStartup.start("spring.context.beandef-registry.post-process")
                    .tag("postProcessor", postProcessor::toString);
                postProcessor.postProcessBeanDefinitionRegistry(registry);//执行后置处理
                postProcessBeanDefRegistry.end();
            }
        }
        ```

        - ConfigurationClassPostProcessor的后置处理方法

          ```java
          public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
              int registryId = System.identityHashCode(registry);
              if (this.registriesPostProcessed.contains(registryId)) {//如果已经执行过了就报错
                  throw new IllegalStateException(
                      "postProcessBeanDefinitionRegistry already called on this post-processor against " + registry);
              }
              if (this.factoriesPostProcessed.contains(registryId)) {
                  throw new IllegalStateException(
                      "postProcessBeanFactory already called on this post-processor against " + registry);
              }
              this.registriesPostProcessed.add(registryId);//设置该后置处理器已执行
          
              processConfigBeanDefinitions(registry);//执行后置处理
          }
          ```

          - processConfigBeanDefinitions方法

            ```java
            public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {//处理配置类BeanDefinition
                List<BeanDefinitionHolder> configCandidates = new ArrayList<>();//用于存放配置类的候选者
                String[] candidateNames = registry.getBeanDefinitionNames();//获取容器中所有BeanDefinititon的名称
            
                for (String beanName : candidateNames) {//遍历所有BeanDefinition
                    BeanDefinition beanDef = registry.getBeanDefinition(beanName);//通过名称获取BeanDefinition
                    if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {//检查该BeanDefinition是否含有CONFIGURATION_CLASS_ATTRIBUTE
                        if (logger.isDebugEnabled()) {
                            logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
                        }
                    }
                    else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {//检查给定的 bean 定义是否是配置类的候选者
                        configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));//如果是则添加至configCandidates中
                    }
                }
            
                // 如果没有Congigutaion候选者，则立即返回
                if (configCandidates.isEmpty()) {
                    return;
                }
            
                // 按先前确定的@Order 值排序（如果适用）
                configCandidates.sort((bd1, bd2) -> {
                    int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
                    int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
                    return Integer.compare(i1, i2);
                });
            
                // 检测通过封闭应用程序上下文提供的任何自定义 bean 名称生成策略
                SingletonBeanRegistry sbr = null;
                if (registry instanceof SingletonBeanRegistry) {//如果是单例注册器则强制转换
                    sbr = (SingletonBeanRegistry) registry;
                    if (!this.localBeanNameGeneratorSet) {
                        BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(
                            AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR);//获取配置类Bean名称生成器
                        if (generator != null) {
                            this.componentScanBeanNameGenerator = generator;//组件扫描器Bean名称生成器
                            this.importBeanNameGenerator = generator;//import注解导入的Bean的名称生成器
                        }
                    }
                }
            
                if (this.environment == null) {
                    this.environment = new StandardEnvironment();
                }
            
                // 解析每个 @Configuration 类
                ConfigurationClassParser parser = new ConfigurationClassParser(
                    this.metadataReaderFactory, this.problemReporter, this.environment,
                    this.resourceLoader, this.componentScanBeanNameGenerator, registry);
            
                Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);//配置类候选者集合从list转为set（当前候选者名单）
                Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());//用于存放已经解析过的配置类
                do {
                    StartupStep processConfig = this.applicationStartup.start("spring.context.config-classes.parse");
                    parser.parse(candidates);//解析所有候选者
                    parser.validate();//校验
            
                    Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());//获取所有被解析的配置类
                    configClasses.removeAll(alreadyParsed);//configClasses移除之前（之前的循环中）已被解析的配置类
            
                    // 读取模型并根据其内容创建 bean 定义
                    if (this.reader == null) {//reader:ConfigurationClassBeanDefinitionReader
                        this.reader = new ConfigurationClassBeanDefinitionReader(
                            registry, this.sourceExtractor, this.resourceLoader, this.environment,
                            this.importBeanNameGenerator, parser.getImportRegistry());
                    }
                    this.reader.loadBeanDefinitions(configClasses);//解析配置类内部注册的Bean
                    alreadyParsed.addAll(configClasses);//添加已经解析的配置类
                    processConfig.tag("classCount", () -> String.valueOf(configClasses.size())).end();
            
                    candidates.clear();//清除候选者集合
                    if (registry.getBeanDefinitionCount() > candidateNames.length) {//如果BeanDefinition的数量大于所有候选者的数量
                        String[] newCandidateNames = registry.getBeanDefinitionNames();//新候选者集合（所有BeanDefinition）
                        Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));//旧候选者集合
                        Set<String> alreadyParsedClasses = new HashSet<>();
                        for (ConfigurationClass configurationClass : alreadyParsed) {//复制一份已被解析的配置类候选者集合
                            alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
                        }
                        for (String candidateName : newCandidateNames) {
                            if (!oldCandidateNames.contains(candidateName)) {//如果旧的候选者集合中没有该Bean（出现这种情况可能是解析配置类的时候解析了其他组件）
                                BeanDefinition bd = registry.getBeanDefinition(candidateName);
                                if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                                    !alreadyParsedClasses.contains(bd.getBeanClassName())) {//检查是否是未被解析的候选者
                                    candidates.add(new BeanDefinitionHolder(bd, candidateName));//添加
                                }
                            }
                        }
                        candidateNames = newCandidateNames;//更新BeanDefinition集合
                    }
                }
                while (!candidates.isEmpty());
            
                // 将 ImportRegistry 注册为 bean 以支持 ImportAware @Configuration 类。ImportRegistry：导入类AnnotationMetadata的注册表。
                if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
                    sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());//使用BeanFatory注册单例Bean
                }
            
                if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
                    // 清除外部提供的 MetadataReaderFactory 中的缓存；这是一个无操作
                    // 对于共享缓存，因为它将被 ApplicationContext 清除。
                    ((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
                }
            }
            ```

            - checkConfigurationClassCandidate方法（检查该类是否能够作为配置类的候选者）
            
              ```java
              public static boolean checkConfigurationClassCandidate(//检查给定的 bean 定义是否是配置类的候选者,metadataReaderFactory – 调用者当前使用的工厂
              			BeanDefinition beanDef, MetadataReaderFactory metadataReaderFactory) {
              
              		String className = beanDef.getBeanClassName();
              		if (className == null || beanDef.getFactoryMethodName() != null) {//如果含有工厂方法或者Bean的Class为null则返回Fasle
              			return false;
              		}
              
              		AnnotationMetadata metadata;//获取元数据
              		if (beanDef instanceof AnnotatedBeanDefinition &&
              				className.equals(((AnnotatedBeanDefinition) beanDef).getMetadata().getClassName())) {
              			//可以重用来自给定 BeanDefinition 的预解析元数据...
              			metadata = ((AnnotatedBeanDefinition) beanDef).getMetadata();
              		}
              		else if (beanDef instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) beanDef).hasBeanClass()) {
              			// 检查已加载的类（如果存在，已加载指的是类加载）...
              			// 因为我们甚至可能无法加载这个类的类文件。
              			Class<?> beanClass = ((AbstractBeanDefinition) beanDef).getBeanClass();
              			if (BeanFactoryPostProcessor.class.isAssignableFrom(beanClass) ||
              					BeanPostProcessor.class.isAssignableFrom(beanClass) ||
              					AopInfrastructureBean.class.isAssignableFrom(beanClass) ||
              					EventListenerFactory.class.isAssignableFrom(beanClass)) {//BeanFactoryPostProcessor、BeanPostProcessor、AOP所用类、事件监听者工厂无法作为配置类
              				return false;
              			}
              			metadata = AnnotationMetadata.introspect(beanClass);//使用StandardAnnotationMetadata包装元数据
              		}
              		else {
              			try {
              				MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(className);
              				metadata = metadataReader.getAnnotationMetadata();
              			}
              			catch (IOException ex) {
              				if (logger.isDebugEnabled()) {
              					logger.debug("Could not find class file for introspecting configuration annotations: " +
              							className, ex);
              				}
              				return false;
              			}
              		}
              
              		Map<String, Object> config = metadata.getAnnotationAttributes(Configuration.class.getName());//获取Configuration注解上的属性
              		if (config != null && !Boolean.FALSE.equals(config.get("proxyBeanMethods"))) {//如果采用proxyBeanMethods：指定是否应该代理@Bean方法以强制执行 bean 生命周期行为，例如即使在用户代码中直接调用@Bean方法的情况下也返回共享的单例 bean 实例。此功能需要方法拦截，通过运行时生成的 CGLIB 子类实现，该子类具有配置类及其方法不允许声明final等限制。
              			beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_FULL);//设置CONFIGURATION_CLASS_ATTRIBUTE为FULL（用于标记该类为配置类）模式
              		}
              		else if (config != null || isConfigurationCandidate(metadata)) {//如果config为空，通过isConfigurationCandidate判断：任何带有Component、ComponentScan、Import、ImportResource注解或者含有Bean注解的都属于配置类候选类
              			beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_LITE);//设置CONFIGURATION_CLASS_ATTRIBUTE为LITE模式
              		}
              		else {
              			return false;
              		}
              
              		// It's a full or lite configuration candidate... Let's determine the order value, if any.这是一个full或lite的配置候选...让我们确定order注解的value，如果有的话。
              		Integer order = getOrder(metadata);//获取该Bean定义的Order属性
              		if (order != null) {
              			beanDef.setAttribute(ORDER_ATTRIBUTE, order);//设置Order属性
              		}
              
              		return true;
              	}
              ```
            
              - Configuration的Full模式和lite模式
            
                - Full模式
            
                  含有Configuration注解的类采用的模式都是Full模式，在Full模式下可以通过以下代码声明Bean的依赖关系
            
                  ```java
                  @Configuration
                  public class ConfigBean2 {
                  
                      @Bean
                      public ServiceA serviceA() {
                          System.out.println("调用serviceA()方法"); //@0
                          return new ServiceA();
                      }
                  
                      @Bean
                      ServiceB serviceB1() {
                          System.out.println("调用serviceB1()方法");
                          ServiceA serviceA = this.serviceA(); //@1
                          return new ServiceB(serviceA);
                      }
                  
                      @Bean
                      ServiceB serviceB2() {
                          System.out.println("调用serviceB2()方法");
                          ServiceA serviceA = this.serviceA(); //@2
                          return new ServiceB(serviceA);
                      }
                  
                  }
                  ————————————————
                  版权声明：本文为CSDN博主「Hell_potato777」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
                  原文链接：https://blog.csdn.net/Hell_potato777/article/details/126800457
                  ```
            
                  此时ServiceB依赖于ServiceA，这是由于在ServiceB的Bean方法中调用了serviceA的Bean方法，只有在full模式下才能有这种Bean的依赖关系
            
                - lite模式
            
                  如果不含有Configuration注解（或者Configuration的proxymode属性为false，springboot中该属性默认为false,但是在spring5中默认为true，为true代表该类可以使用CGlib进行代理），但是含有Component注解等，或者只含有Bean注解的类同样被加载为配置类，只是采用lite模式，lite模式无法声明Bean的依赖关系，在以上代码中serviceA方法将会调用三次，三次都产生不同的ServiceA，以下是一个lite配置类的案例
            
                  ```java
                  @Component
                  public class liteConfig {
                      @Bean
                      public Bean03 bean03()
                      {
                          return new Bean03();
                      }
                  
                  }
                  
                  ```
            
                  测试代码
            
                  ```java
                  public class IOCMain {
                  
                      public static void main(String[] args) {
                          ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
                          System.out.println("容器中的Bean3（第一次获取）："+context.getBean(Bean03.class));
                          System.out.println("容器中的Bean3（第二次获取）："+context.getBean(Bean03.class));
                          liteConfig liteConfig = context.getBean(liteConfig.class);
                          System.out.println("容器中的liteconfig："+liteConfig);
                          System.out.println("调用liteconfig中的Bean03方法（第一次调用）："+liteConfig.bean03());
                          System.out.println("调用liteconfig中的Bean03方法（第二次调用）："+liteConfig.bean03());
                      }
                  }
                  ```
            
                  执行结果
            
                  ```java
                  - - - - - - - - - 
                      容器中的Bean3（第一次获取）：com.spring.IOC.POJO.Bean03@42a15bdc
                      容器中的Bean3（第二次获取）：com.spring.IOC.POJO.Bean03@42a15bdc
                      容器中的liteconfig：com.spring.IOC.POJO.liteConfig@44a59da3
                      调用liteconfig中的Bean03方法（第一次调用）：com.spring.IOC.POJO.Bean03@27e47833
                      调用liteconfig中的Bean03方法（第二次调用）：com.spring.IOC.POJO.Bean03@6f6745d6
                  
                  ```
            
                  这说明容器中的Bean3仍旧是单例但是lite的配置类中的方法还是普通方法并没有加强，但是如果采用configuration注解结果有不一样
            
                  ```java
                  容器中的Bean3（第一次获取）：com.spring.IOC.POJO.Bean03@3ddc6915
                  容器中的Bean3（第二次获取）：com.spring.IOC.POJO.Bean03@3ddc6915
                  容器中的liteconfig：com.spring.IOC.POJO.liteConfig$$EnhancerBySpringCGLIB$$35584b10@704deff2
                  调用liteconfig中的Bean03方法（第一次调用）：com.spring.IOC.POJO.Bean03@3ddc6915
                  调用liteconfig中的Bean03方法（第二次调用）：com.spring.IOC.POJO.Bean03@3ddc6915
                  ```
            
            - ConfigurationClassParser解析配置类候选者
            
              ```java
              parser.parse(candidates);//解析所有候选者
              
              public void parse(Set<BeanDefinitionHolder> configCandidates) {
                  for (BeanDefinitionHolder holder : configCandidates) {//遍历集合根据Beandefinition类型解析
                      BeanDefinition bd = holder.getBeanDefinition();
                      try {//根据BeanDefinition类型解析
                          if (bd instanceof AnnotatedBeanDefinition) {
                              parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
                          }
                          else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
                              parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
                          }
                          else {
                              parse(bd.getBeanClassName(), holder.getBeanName());
                          }
                      }
                      catch (BeanDefinitionStoreException ex) {
                          throw ex;
                      }
                      catch (Throwable ex) {
                          throw new BeanDefinitionStoreException(
                              "Failed to parse configuration class [" + bd.getBeanClassName() + "]", ex);
                      }
                  }
              
                  this.deferredImportSelectorHandler.process();
              }
              ```
            
              - AnnotatedBeanDefinition的解析方法（parse）
            
                ```java
                protected final void parse(AnnotationMetadata metadata, String beanName) throws IOException {
                    processConfigurationClass(new ConfigurationClass(metadata, beanName), DEFAULT_EXCLUSION_FILTER);
                }
                
                protected void processConfigurationClass(ConfigurationClass configClass, Predicate<String> filter) throws IOException {
                    if (this.conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {//condition注解解析
                        return;
                    }
                
                    ConfigurationClass existingClass = this.configurationClasses.get(configClass);//从configurationClasses获取该配置类的ConfigurationClass
                    if (existingClass != null) {//如果不为空则代表可能从其他地方导入了配置类
                        if (configClass.isImported()) {//是否是import导入的配置类
                            if (existingClass.isImported()) {
                                existingClass.mergeImportedBy(configClass);//添加该配置类
                            }
                            // 否则忽略新导入的配置类；现有的非导入类会覆盖它。
                            return;
                        }
                        else {
                            // 找到显式 bean 定义，可能替换导入。
                            // 让我们删除旧的并使用新的。
                            this.configurationClasses.remove(configClass);//删除该configClass
                            this.knownSuperclasses.values().removeIf(configClass::equals);//从knownSuperclasses中删除
                        }
                    }
                
                    // 递归处理配置类及其超类层次结构。
                    SourceClass sourceClass = asSourceClass(configClass, filter);//使用SourceClass包装Bean的Class
                    do {
                        sourceClass = doProcessConfigurationClass(configClass, sourceClass, filter);//解析配置类（如果该配置类含有父类则会返回父类的sourceClass，没有父类则返回空）
                    }
                    while (sourceClass != null);
                
                    this.configurationClasses.put(configClass, configClass);//在配置类集合中添加该类
                }
                ```
            
                