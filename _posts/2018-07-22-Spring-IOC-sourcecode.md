---
layout: post
title: Spring IOC 源码学习
categories: 框架
description: Spring
keywords: IOC
---

学习Spring3的源码。

## IOC部分

一般我们都是这样开始使用Spring的。

```java
ApplicationContext context = new FileSystemXmlApplicationContext
                ("src/Beans.xml");
        HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
        obj.getMessage();
```

那我们就从这里开始分析。

`FileSystemXmlApplicationContext`可以加载配置文件，生成bean容器。从而我们可以从容器中获取bean，使用bean。

> Spring提供了以下两种不同类型的容器。
**Spring BeanFactory 容器。**
它是最简单的容器，给 DI 提供了基本的支持，它用 org.springframework.beans.factory.BeanFactory 接口来定义。
**Spring ApplicationContext 容器**
该容器添加了更多的企业特定的功能，例如从一个属性文件中解析文本信息的能力，发布应用程序事件给感兴趣的事件监听器的能力。该容器是由 org.springframework.context.ApplicationContext 接口定义。

ApplicationContext 容器包括 BeanFactory 容器的所有功能，所以通常建议使用ApplicationContext容器 。这里我们使用的也是ApplicationContext 容器。

**上面提到的两种容器都实现了BeanFactory接口。**

所以我们先看下BeanFactory接口的定义。

```java
public interface BeanFactory {
    String FACTORY_BEAN_PREFIX = "&";

    Object getBean(String var1) throws BeansException;

    <T> T getBean(String var1, Class<T> var2) throws BeansException;

    <T> T getBean(Class<T> var1) throws BeansException;

    Object getBean(String var1, Object... var2) throws BeansException;

    boolean containsBean(String var1);

    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;

    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, Class var2) throws NoSuchBeanDefinitionException;

    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;

    String[] getAliases(String var1);
}
```

这个接口是所有IOC容器的祖宗，各种IOC容器都只是它的实现或者为了满足特别需求的扩展实现，包括我们平时用的最多的ApplicationContext。从上面的方法就可以看出，这些工厂的实现最大的作用就是根据bean的名称或类型等，返回一个bean的实例。

因为在Spring中bean的实例化是可以延迟加载的，所以我们无法直接的创建一个Map<String,Object>来创建这些bean的实例，但我们可以通过持有bean的定义来让bean在需要的时候实例化。

我们保存的是bean的定义。所以需要相应的接口。

下面是BeanDefinition接口。

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
    String SCOPE_SINGLETON = "singleton";
    String SCOPE_PROTOTYPE = "prototype";
    int ROLE_APPLICATION = 0;
    int ROLE_SUPPORT = 1;
    int ROLE_INFRASTRUCTURE = 2;

    String getParentName();

    void setParentName(String var1);

    String getBeanClassName();

    void setBeanClassName(String var1);

    String getFactoryBeanName();

    void setFactoryBeanName(String var1);

    String getFactoryMethodName();

    void setFactoryMethodName(String var1);

    String getScope();

    void setScope(String var1);

    boolean isLazyInit();

    void setLazyInit(boolean var1);

    String[] getDependsOn();

    void setDependsOn(String[] var1);

    boolean isAutowireCandidate();

    void setAutowireCandidate(boolean var1);

    boolean isPrimary();

    void setPrimary(boolean var1);

    ConstructorArgumentValues getConstructorArgumentValues();

    MutablePropertyValues getPropertyValues();

    boolean isSingleton();

    boolean isPrototype();

    boolean isAbstract();

    int getRole();

    String getDescription();

    String getResourceDescription();

    BeanDefinition getOriginatingBeanDefinition();
}
```

通过上面两个接口，我们就应该了解到，IOC容器持有一个Map<String,BeanDefinition>，这样在任何时候我们想用哪个bean，就去取到它的bean定义，然后我们就可以创造出一个bean实例。

上面提到的Map是在DefaultListableBeanFactory这个类里面定义的。我们看下这个类，这个类里面属性和方法很多，但我们现在只需要了解其中一个Map的定义即可。


```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
...
/** Map of bean definition objects, keyed by bean name */
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap();
...
}
```

可以看到这个Map的key是bean name，value是bean的定义。

接下来我们捋一下思路。

从现在来看，我们需要什么才能把Map填充呢？也就是初始化bean工厂呢，或者说建立IOC容器。

1. 需要一个File指向我们的XML文件（本文的配置文件都已XML为例，因为这是我们最熟悉的），专业点可以叫资源定位，简单点可以说我们需要一些工具来完成找到XML文件的所在位置。

2. 需要一个Reader来读取我们的XML文件，专业点叫DOM解析，简单点说，就是把XML文件的各种定义都给拿出来。

3. 需要将读出来的数据都设置到Map当中。

在本文的第一段代码中我们是这样初始化容器的。`ApplicationContext context = new FileSystemXmlApplicationContext
                ("src/Beans.xml");`

实际上这一行代码包括了我们上面提到的三个步骤。

如果按照三个步骤来写代码的话，应该是这样子的。

```java
ClassPathResource classPathResource = new ClassPathResource("beans.xml");
        DefaultListableBeanFactory defaultListableBeanFactory = new DefaultListableBeanFactory();
        XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(defaultListableBeanFactory);
        xmlBeanDefinitionReader.loadBeanDefinitions(classPathResource);
        ((Person)defaultListableBeanFactory.getBean("person")).work();
```

Person类。

```java
public class Person {

    public void work(){
        System.out.println("I am working");
    }
}
```

beans.xml。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
 <bean id="person" class="test.Person"></bean>
</beans>
```

然后运行代码，结果与我们期望的是一样的，成功的解析了XML文件，并注册了一个bean定义，而且我们使用getBean方法也成功得到了Person的实例。

上述这段程序当中可以看出，bean工厂的初始化一共使用了四行程序。

我们再分析一下那几步初始化容器具体做了什么。

1. 第一行完成了我们的第一步，即资源定位，采用classpath定位，因为我的beans.xml文件是放在src下面的。

2. 第二行创建了一个默认的bean工厂。

3. 第三行创建了一个reader，从名字就不难看出，这个reader是用来读取XML文件的。这一步要多说一句，其中将我们创建的defaultListableBeanFactory作为参数传给了reader的构造函数，这里是为了第四步读取XML文件做准备。

4. 第四行使用reader解析XML文件，并将读取的bean定义回调设置到defaultListableBeanFactory当中。其实回调这一步就相当于我们上述的注册这一步。

5. 这个时候defaultListableBeanFactory已经被正确初始化了，我们已经可以使用它的一些方法了，比如上面所使用的获取bean个数，以及获得一个bean实例的方法。

这里是我们按照自己的思路手动去初始化容器。那接下来看看`ApplicationContext context = new FileSystemXmlApplicationContext
                ("src/Beans.xml");`这行代码中Spring是如何做的。

首先看下`FileSystemXmlApplicationContext`的构造函数。它有多个构造函数，但最终都是调用的下面这个。

```java
public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {
        super(parent);
        this.setConfigLocations(configLocations);
        if(refresh) {
            this.refresh();
        }

    }
```

里面的`refresh`方法就是初始化容器的入口。

```java
public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {
            this.prepareRefresh();
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                this.invokeBeanFactoryPostProcessors(beanFactory);
                this.registerBeanPostProcessors(beanFactory);
                this.initMessageSource();
                this.initApplicationEventMulticaster();
                this.onRefresh();
                this.registerListeners();
                this.finishBeanFactoryInitialization(beanFactory);
                this.finishRefresh();
            } catch (BeansException var4) {
                this.destroyBeans();
                this.cancelRefresh(var4);
                throw var4;
            }

        }
    }
```

 这里面列出了IOC容器初始化的大致步骤，第一步很容易看出来是初始化准备，这个方法里只是设置了一个活动标识，我们主要来看第二步，obtainFreshBeanFactory这个方法，它是用来告诉子类刷新内部的bean工厂，接下来我们跟踪进去看看。

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
        this.refreshBeanFactory();
        ConfigurableListableBeanFactory beanFactory = this.getBeanFactory();
        if(this.logger.isDebugEnabled()) {
            this.logger.debug("Bean factory for " + this.getDisplayName() + ": " + beanFactory);
        }

        return beanFactory;
    }
```

该方法中第一句便调用了另外一个`refreshBeanFactory`方法，这个方法是`AbstractApplicationContext`中的抽象方法，具体的实现并没有在这个抽象类中实现，而是留给了子类，我们追踪到这个子类当中去看一下。该方法在子类`AbstractRefreshableApplicationContext`中实现，我们来看。

```java
protected final void refreshBeanFactory() throws BeansException {
        if(this.hasBeanFactory()) {
            this.destroyBeans();
            this.closeBeanFactory();
        }

        try {
            DefaultListableBeanFactory ex = this.createBeanFactory();
            ex.setSerializationId(this.getId());
            this.customizeBeanFactory(ex);
            this.loadBeanDefinitions(ex);
            Object var2 = this.beanFactoryMonitor;
            synchronized(this.beanFactoryMonitor) {
                this.beanFactory = ex;
            }
        } catch (IOException var4) {
            throw new ApplicationContextException("I/O error parsing bean definition source for " + this.getDisplayName(), var4);
        }
    }
```

方法加上了final关键字，也就是说此方法不可被重写，可以很清楚的看到，IOC容器的初始化就是在这个方法里发生的，第一步先是判断有无现有的工厂，有的话便会将其摧毁，否则，就会创建一个默认的bean工厂，也就是前面提到的`DefaultListableBeanFactory`，注意看`loadBeanDefinitions(beanFactory)`，这里，当我们创建了一个默认的bean工厂以后，便是载入bean的定义。这与我们上一章所使用的原始的创建bean工厂的方式极为相似。

看到这里，其实不难看出，`FileSystemXmlApplicationContext`的初始化方法中，其实已经包含了我们上一章当中原始的创建过程，这个类是一个现有的，spring已经为我们实现好的BeanFactory的实现类。在项目当中，我们经常会用到。

接下来我们再看`loadBeanDefinitions`，这个方法是由AbstractXmlApplicationContext抽象类实现的。

```java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
        beanDefinitionReader.setResourceLoader(this);
        beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));
        this.initBeanDefinitionReader(beanDefinitionReader);
        this.loadBeanDefinitions(beanDefinitionReader);
    }
```

 第一行首先定义了一个reader，很明显，这个就是spring为读取XML配置文件而定制的读取工具，这里·AbstractXmlApplicationContext·间接实现了·ResourceLoader·接口，所以该方法的第二行才得以成立，最后一行便是真正载入bean定义的过程。我们追踪其根源，可以发现最终的读取过程正是由reader完成的，代码如下。

```java
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
        Assert.notNull(encodedResource, "EncodedResource must not be null");
        if (logger.isInfoEnabled()) {
            logger.info("Loading XML bean definitions from " + encodedResource.getResource());
        }

        Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
        if (currentResources == null) {
            currentResources = new HashSet<EncodedResource>(4);
            this.resourcesCurrentlyBeingLoaded.set(currentResources);
        }
        if (!currentResources.add(encodedResource)) {
            throw new BeanDefinitionStoreException(
                    "Detected cyclic loading of " + encodedResource + " - check your import definitions!");
        }
        try {
            InputStream inputStream = encodedResource.getResource().getInputStream();
            try {
                InputSource inputSource = new InputSource(inputStream);
                if (encodedResource.getEncoding() != null) {
                    inputSource.setEncoding(encodedResource.getEncoding());
                }
                return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
            }
            finally {
                inputStream.close();
            }
        }
        catch (IOException ex) {
            throw new BeanDefinitionStoreException(
                    "IOException parsing XML document from " + encodedResource.getResource(), ex);
        }
        finally {
            currentResources.remove(encodedResource);
            if (currentResources.isEmpty()) {
                this.resourcesCurrentlyBeingLoaded.remove();
            }
        }
    }
```

这个方法中不难发现，try块中的代码才是载入bean定义的真正过程，spring将资源返回的输入流包装以后传给了`doLoadBeanDefinitions`方法，我们进去看看发生了什么。

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
            throws BeanDefinitionStoreException {
        try {
            int validationMode = getValidationModeForResource(resource);
            Document doc = this.documentLoader.loadDocument(
                    inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
            return registerBeanDefinitions(doc, resource);
        }
        catch (BeanDefinitionStoreException ex) {
            throw ex;
        }
        catch (SAXParseException ex) {
            throw new XmlBeanDefinitionStoreException(resource.getDescription(),
                    "Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
        }
        catch (SAXException ex) {
            throw new XmlBeanDefinitionStoreException(resource.getDescription(),
                    "XML document from " + resource + " is invalid", ex);
        }
        catch (ParserConfigurationException ex) {
            throw new BeanDefinitionStoreException(resource.getDescription(),
                    "Parser configuration exception parsing XML from " + resource, ex);
        }
        catch (IOException ex) {
            throw new BeanDefinitionStoreException(resource.getDescription(),
                    "IOException parsing XML document from " + resource, ex);
        }
        catch (Throwable ex) {
            throw new BeanDefinitionStoreException(resource.getDescription(),
                    "Unexpected exception parsing XML document from " + resource, ex);
        }
    }
```

可以看到，Spring采用documentLoader将资源转换成了Document接口，从这里就开始进行XML的解析了。

最后再看一下`registerBeanDefinitions`方法，这里面调用了`DefaultBeanDefinitionDocumentReader`的`parseBeanDefinitions`方法。

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
        if (delegate.isDefaultNamespace(root)) {
            NodeList nl = root.getChildNodes();
            for (int i = 0; i < nl.getLength(); i++) {
                Node node = nl.item(i);
                if (node instanceof Element) {
                    Element ele = (Element) node;
                    if (delegate.isDefaultNamespace(ele)) {
                        parseDefaultElement(ele, delegate);
                    }
                    else {
                        delegate.parseCustomElement(ele);
                    }
                }
            }
        }
        else {
            delegate.parseCustomElement(root);
        }
    }
```

 这里分了两种解析路线，一个是默认的，一个是自定义的，从这里我们可以看出，我们是可以在Spring的配置文件中自定义节点的。

再往下走就基本上到了Spring针对具体标签解析的过程。

到这对Spring的IOC部分源码的分析就告一段落。

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
