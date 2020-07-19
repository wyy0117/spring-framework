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
