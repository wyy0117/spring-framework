## 入口
```
org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(org.springframework.core.io.Resource)
```
```
org.springframework.beans.factory.support.AbstractBeanFactory.getBean(java.lang.String)
```
## 流程
1. 读取解析配置文件
    ```
   org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(org.springframework.core.io.Resource)
   ```
   1. encoding
        1. 设置配置文件encoding和charset，把Resource转为EncodedResource
   1. EncodedResource转为InputSource
        1. 取出inputStream并设置encoding
   1. 解析出Document
      ```
      org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadDocument
      ```
      1. 初始化EntityResolver
         ```
         org.springframework.beans.factory.xml.XmlBeanDefinitionReader.getEntityResolver
         ```
         主要是对xml的声明定义做检查，可以直接在本地资源中检查DTD(Document Type Definition)和XSD(Xml Schema Definition)
      1. 获取验证模式  
          校验是DTD还是XSD，默认的获取验证模式的方式就是判断文件中是否包含DOCTYPE，如果包含就是DTD否则就是XSD
      1. 解析出Document
1. 注册BeanDefinition
    ```
   org.springframework.beans.factory.xml.XmlBeanDefinitionReader.registerBeanDefinitions
   ```
    1. 创建BeanDefinitionDocumentReader
        ```
       org.springframework.beans.factory.xml.XmlBeanDefinitionReader.createBeanDefinitionDocumentReader
       ```  
    1. 解析Document，注册BeanDefinition
        ```
       org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.registerBeanDefinitions
       ```
       1. 创建代理类BeanDefinitionParserDelegate
            ```
          org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.createDelegate
          ```
       1. 解析profile属性，判断与当前运行的profile是否需要该profile
       1. 处理xml解析之前要做什么事
            ```
          org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.preProcessXml
          ```
          模板方法，默认该方法为空方法，子类可以做自定义
       1. 解析xml
            获取跟节点的namespace，判断是否是spring默认的namespace  
            1. 如果是默认的namespace，解析出所有的子节点
                1. 如果是默认的namespace，直接解析
                    ```
                    org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.parseDefaultElement
                    ```  
                   1. 解析<import>
                        ```
                      org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.importBeanDefinitionResource
                      ```
                   1. 解析<alias>
                        ```
                      org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.processAliasRegistration
                      ```
                   1. 解析<bean>
                        ```
                      org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.processBeanDefinition
                      ```
                        1. 解析BeanDefinition
                            ```
                           org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseBeanDefinitionElement(org.w3c.dom.Element)
                           ```
                           1. 解析出id和name（aliases）
                           1. check unique
                           1. 解析出AbstractBeanDefinition
                                ```
                              org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseBeanDefinitionElement(org.w3c.dom.Element, java.lang.String, org.springframework.beans.factory.config.BeanDefinition)
                              ```
                                1. 创建AbstractBeanDefinition
                                    ```
                                   org.springframework.beans.factory.support.BeanDefinitionReaderUtils.createBeanDefinition
                                   ```
                                1. 解析scope等属性
                                    ```
                                   org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseBeanDefinitionAttributes
                                   ```
                                1. 解析原数据<meta>
                                    ```
                                   org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseMetaElements
                                   ```
                                1. 解析<look-up>
                                    ```
                                   org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseLookupOverrideSubElements
                                   ```
                                1. 解析<replaced-method>
                                    ```
                                   org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseReplacedMethodSubElements
                                   ```
                                1. 解析构造器
                                    ```
                                   org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseConstructorArgElements
                                   ```
                                1. 解析<property>
                                    ```
                                   org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parsePropertyElements
                                   ```
                                1. 解析<qualifier/>
                                    ```
                                   org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseQualifierElements
                                   ```
                             1. 创建BeanDefinitionHolder
                        1. 装饰BeanDefinition
                            ```
                           org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.decorateBeanDefinitionIfRequired(org.w3c.dom.Element, org.springframework.beans.factory.config.BeanDefinitionHolder)
                           ```
                        1. 注册BeanDefinition
                            ```
                           org.springframework.beans.factory.support.BeanDefinitionReaderUtils.registerBeanDefinition
                           ```
                        1. 发布注册事件
                            ```
                           org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.getReaderContext
                           ```
                           ```
                           org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.getReaderContext
                           ```
                   1. 解析<beans>
                        ```
                      org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.doRegisterBeanDefinitions
                      ```
                1. 否则使用代理类自定义解析
                    ```
                    org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseCustomElement(org.w3c.dom.Element)
                    ```  
            1. 否则自定义解析Element
       1. 处理解析xml之后要做什么事
            ```
          org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.postProcessXml
          ```
1. 初始化bean
    1. 转换对应beanName
        ```
       org.springframework.beans.factory.support.AbstractBeanFactory.transformedBeanName
       ```
       主要就是去除beanName前缀&
    1. 尝试从缓存中加载单例
        ```
        org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(java.lang.String)        
       ```
        此时获取到的有可能是bean实例也可能是FactoryBean  
        1. 处理FactoryBean
            ```
            org.springframework.beans.factory.support.AbstractBeanFactory.getObjectForBeanInstance
            ```
            1. 根据name校验是否是真的FactoryBean
            1. 从缓存中获取
                ```
               org.springframework.beans.factory.support.FactoryBeanRegistrySupport.getCachedObjectForFactoryBean
               ```
            1. 如果缓存中没有，将GenericBeanDefinition转换为RootBeanDefinition
                ```
               org.springframework.beans.factory.support.AbstractBeanFactory.getMergedLocalBeanDefinition
               ```     
            1. 从FactoryBean中获取bean对象
                ```
               org.springframework.beans.factory.support.FactoryBeanRegistrySupport.getObjectFromFactoryBean
               ```
               ```
               org.springframework.beans.factory.support.FactoryBeanRegistrySupport.doGetObjectFromFactoryBean
               ```
               从缓存中取，如果取不到，有后处理  
               1. 创建之前  
                   标记正在创建
                    ```
                  org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.beforeSingletonCreation
                  ```
               1. 应用后处理器
                    ```
                  org.springframework.beans.factory.support.FactoryBeanRegistrySupport.postProcessObjectFromFactoryBean
                  ```
               1. 创建后
                    ```
                  org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.afterSingletonCreation
                  ```   
    1. 从parentBeanFactory中获取
    1. 将存储xml配置文件的GenericBeanDefinition转换为RootBeanDefinition
        ```
       org.springframework.beans.factory.support.AbstractBeanFactory.getMergedLocalBeanDefinition
       ```
    1. 缓存依赖
        ```
       org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.registerDependentBean
       ```
    1. 针对不同的scope进行bean创建
        1. 如果是单例
            ```
           org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(java.lang.String, org.springframework.beans.factory.ObjectFactory<?>)
           ```
           如果有异常，需要destroy
           ```
           org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroySingleton
           ```
           1. 尝试从缓存中取
           1. 创建之前
                ```
              org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.beforeSingletonCreation
              ```
           1. 从ObjectFactory中获取，即调用getObject（）方法
                ```
              org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
              ```
              1. 从RootBeanDefinition解析出class
                    ```
                 org.springframework.beans.factory.support.AbstractBeanFactory.resolveBeanClass
                 ```
              1. 准备method override
                    ```
                 org.springframework.beans.factory.support.AbstractBeanDefinition.prepareMethodOverrides
                 ```
              1. 处理实例化前
                 ```
                 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.resolveBeforeInstantiation
                 ```
              1. 创建bean
                 ```
                 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean
                 ```
                 1. 创建bean实例
                    ```
                    org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance
                    ```
                 1. 将ObjectFactory加入缓存
                 1. 填充属性
                    ```
                    org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean
                    ```
                 1. 调用定义的初始化方法，初始化bean
                    ```
                    org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
                    ```
                    1. 感知注入
                        ```
                       org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeAwareMethods
                       ```
                    1. 应用前处理器
                        ```
                       org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInitialization
                       ```
                    1. 调动init方法
                       ```
                       org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods
                       ```
                       1. 如果是InitializingBean类型，先调用afterPropertiesSet
                            ```
                          org.springframework.beans.factory.InitializingBean.afterPropertiesSet
                          ```
                       1. 调用用户自定义init方法
                            ```
                          org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeCustomInitMethod
                          ``` 
                    1. 应用后处理器
                        ```
                       org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsAfterInitialization
                       ```
                 1. 处理循环依赖
                 1. 注册销毁方法的扩展
                    ```
                    org.springframework.beans.factory.support.AbstractBeanFactory.registerDisposableBeanIfNecessary
                    ```
           1. 创建后
                ```
              org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.afterSingletonCreation
              ```
           1. 加入缓存
                ```
              org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.addSingleton
              ```
        1. 如果是prototype
             1. 创建之前
                ```
                org.springframework.beans.factory.support.AbstractBeanFactory.beforePrototypeCreation
                ```
             1. 创建bean
                ```
                org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
                ```
             1. 创建后
                ```
                org.springframework.beans.factory.support.AbstractBeanFactory.afterPrototypeCreation
                ```
             1. 获取bean实例
                ```
                org.springframework.beans.factory.support.AbstractBeanFactory.getObjectForBeanInstance
                ```
        1. 指定scope
            1. 获取Scope
            1. 创建之前
                ```
               org.springframework.beans.factory.support.AbstractBeanFactory.beforePrototypeCreation
               ```
            1. 创建bean
                ```
               org.springframework.beans.factory.support.AbstractBeanFactory.createBean
               ```
            1. 创建之后
                ```
               org.springframework.beans.factory.support.AbstractBeanFactory.afterPrototypeCreation
               ```
            1. 获取bean对象
                ```
               org.springframework.beans.factory.support.AbstractBeanFactory.getObjectForBeanInstance
               ```
        1. 处理失败
            ```
           org.springframework.beans.factory.support.AbstractBeanFactory.cleanupAfterBeanCreationFailure
           ```
    1. 类型转换 
        ```
           org.springframework.beans.TypeConverter.convertIfNecessary(java.lang.Object, java.lang.Class<T>)
       ```
