---
layout: post
title: 设计一个IOC
categories: Java
description: IOC
keywords: IOC
---

## 概述

> 在我们设计一个IOC之前需要对IOC的概念和Java反射的原理有一定的了解。

我们要自己设计一个IOC，那么目标是什么呢？ 我们的IOC容器要可以存储对象，还要有注解注入的功能即可。

Java语言允许通过程序化的方式间接对Class进行操作，Class文件由**类装载器**装载后，在JVM中将形成一份描述Class结构的元信息对象，通过该元信息对象可以获知Class的结构信息：如**构造函数**、**属性**和**方法**等。

Java允许用户借由这个Class相关的元信息对象间接调用Class对象的功能，这就为使用程序化方式操作Class对象开辟了途径。

## 设计IOC

我们将从一个简单例子开始探访Java反射机制的征程，下面的**Hero类**拥有一个构造函数、五个方法以及两个属性，如代码清单所示：

```java
/**
 * LOL英雄
 */
public class Hero {

	// 英雄名称
	private String name;
	// 装备名称
	private String outfit;

	public Hero() {

	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getOutfit() {
		return outfit;
	}

	public void setOutfit(String outfit) {
		this.outfit = outfit;
	}
	
	public void say(){
		System.out.println(name + "购买了" + outfit);
	}
}
```

测试代码：

```java
public class Test {

	public static void main(String[] args) throws Exception {
		//1. 通过类装载器获取Hero类对象  
		ClassLoader loader = Thread.currentThread().getContextClassLoader();   
		Class<?> clazz = loader.loadClass("com.biezhi.ioc.Hero");   
		  
		//2. 获取类的默认构造器对象并通过它实例化Hero  
		Constructor<?> cons = clazz.getDeclaredConstructor((Class[])null);   
		Hero hero = (Hero)cons.newInstance();  
		
		//3. 通过反射方法设置属性  
		Method setBrand = clazz.getMethod("setName", String.class);
		setBrand.invoke(hero, "小鱼人");
		Method setColor = clazz.getMethod("setOutfit", String.class);
		setColor.invoke(hero, "爆裂魔杖");
		
		// 4. 运行方法
		hero.say();
	}
}
```

输出：小鱼人购买了爆裂魔杖

这说明我们完全可以通过编程方式调用Class的各项功能，这和直接通过构造函数和方法调用类功能的效果是一致的，只不过前者是间接调用，后者是直接调用罢了。

简单的例子说完了，我们开始设计一个自己的IOC容器。

### Container接口

首先设计接口，一个IOC容器中最核心的当属容器接口，来一个Container。

那么容器里应该有什么呢，我想它至少要有存储和移除一个对象的能力，其次可以含括更多的获取和注册对象的方法。

```java
public interface Container {

	/**
	 * 根据Class获取Bean
	 * @param clazz
	 * @return
	 */
	public <T> T getBean(Class<T> clazz);
	
	/**
	 * 根据名称获取Bean
	 * @param name
	 * @return
	 */
	public <T> T getBeanByName(String name);
	
	/**
	 * 注册一个Bean到容器中
	 * @param object
	 */
	public Object registerBean(Object bean);
	
	/**
	 * 注册一个Class到容器中
	 * @param clazz
	 */
	public Object registerBean(Class<?> clazz);
	
	/**
	 * 注册一个带名称的Bean到容器中
	 * @param name
	 * @param bean
	 */
	public Object registerBean(String name, Object bean);
	
	/**
	 * 删除一个bean
	 * @param clazz
	 */
	public void remove(Class<?> clazz);
	
	/**
	 * 根据名称删除一个bean
	 * @param name
	 */
	public void removeByName(String name);
	
	/**
	 * @return	返回所有bean对象名称
	 */
	public Set<String> getBeanNames();
	
	/**
	 * 初始化装配
	 */
	public void initWired();
}
```

### Container实现

接口的实现代码：

```java
@SuppressWarnings("unchecked")
public class SampleContainer implements Container {
	
	/**
     * 保存所有bean对象，格式为 com.xxx.Person : @52x2xa
     */
    private Map<String, Object> beans;
    
    /**
     * 存储bean和name的关系
     */
    private Map<String, String> beanKeys;
    
    public SampleContainer() {
    	this.beans = new ConcurrentHashMap<String, Object>();
    	this.beanKeys = new ConcurrentHashMap<String, String>();
    }
	
	@Override
	public <T> T getBean(Class<T> clazz) {
		String name = clazz.getName();
		Object object = beans.get(name);
		if(null != object){
			return (T) object;
		}
		return null;
	}

	@Override
	public <T> T getBeanByName(String name) {
		String className = beankeys.get(name);
		Object obj = beans.get(className);
		if(null != object){
			return (T) object;
		}
		return null;
	}

	@Override
	public Object registerBean(Object bean) {
		String name = bean.getClass().getName();
		beanKeys.put(name, name);
		beans.put(name, bean);
		return bean;
	}
	
	@Override
	public Object registerBean(Class<?> clazz) {
		String name = clazz.getName();
		beanKeys.put(name, name);
		//ReflectUtil,自己定义的反射工具类，源码在GitHub。
		Object bean = ReflectUtil.newInstance(clazz);
		beans.put(name, bean);
		return bean;
	}

	@Override
	public Object registerBean(String name, Object bean) {
		String className = bean.getClass().getName();
		beanKeys.put(name, className);
		beans.put(className, bean);
		return bean;
	}

	@Override
	public Set<String> getBeanNames() {
		return beanKeys.keySet();
	}

	@Override
	public void remove(Class<?> clazz) {
		String className = clazz.getName();
		if(null != className && !className.equals("")){
			beanKeys.remove(className);
			beans.remove(className);
		}
	}

	@Override
	public void removeByName(String name) {
		String className = beanKeys.get(name);
		if(null != className && !className.equals("")){
			beanKeys.remove(name);
			beans.remove(className);
		}
	}

	@Override
	public void initWired() {
		Iterator<Entry<String, Object>> it = beans.entrySet().iterator();
        while (it.hasNext()) {
			Map.Entry<String, Object> entry = (Map.Entry<String, Object>) it.next();
			Object object = entry.getValue();
			injection(object);
		}
	}

	/**
	 * 注入对象
	 * @param object
	 */
	public void injection(Object object) {
		// 所有字段
	    try {
			Field[] fields = object.getClass().getDeclaredFields();
			for (Field field : fields) {
				// 需要注入的字段，AutoWired是我们自己定义的注解，源码可在GitHub找到。
				AutoWired autoWired = field.getAnnotation(autoWired.class);
			    if (null != autoWired) {
			    	
			    	// 要注入的字段
			        Object autoWiredField = null;
			        
			        String name = autoWired.name();
	        		if(!name.equals("")){
	        			String className = beanKeys.get(name);
	        			if(null != className && !className.equals("")){
	        				autoWiredField = beans.get(className);
	        			}
	        			if (null == autoWiredField) {
				            throw new RuntimeException("Unable to load " + name);
				        }
	        		} else {
	        			if(autoWired.value() == Class.class){
	        				autoWiredField = recursiveAssembly(field.getType());
				        } else {
				        	// 指定装配的类
				    		autoWiredField = this.getBean(autoWired.value());
				            if (null == autoWiredField) {
				            	autoWiredField = recursiveAssembly(autoWired.value());
				            }
						}
					}
	        		
			        if (null == autoWiredField) {
			            throw new RuntimeException("Unable to load " + field.getType().getCanonicalName());
			        }
			        
			        boolean accessible = field.isAccessible();
			        field.setAccessible(true);
			        field.set(object, autoWiredField);
			        field.setAccessible(accessible);
			    }
			}
		} catch (SecurityException e) {
        	e.printStackTrace();
        } catch (IllegalArgumentException e) {
        	e.printStackTrace();
        } catch (IllegalAccessException e) {
        	e.printStackTrace();
        }
	}
	
	private Object recursiveAssembly(Class<?> clazz){
    	if(null != clazz){
    		return this.registerBean(clazz);
    	}
    	return null;
    }

}
```

这里将所有Bean的名称存储在 beanKeys 这个map中，将所有的对象存储在 beans 中，用 beanKeys 维护名称和对象的关系。

在装配的时候步骤如下：

* 1.判断是否使用了自定义命名的对象（是：根据name查找bean）
* 2.判断是否使用了Class类型Bean（是：根据Class查找Bean，如果查找不到则创建一个无参构造函数的Bean）

### 测试

```java
public class IocTest {

	private static Container container = new SampleContainer();
	
	public static void baseTest(){
		container.registerBean(Lol.class);
		// 初始化注入（注入字段对象）
		container.initWired();
		
		//Lol,Lol2,Lol3源码可在GitHub找到，此处不在展示
		Lol lol = container.getBean(Lol.class);
		lol.work();
	}
	
	public static void iocClassTest(){
		container.registerBean(Lol2.class);
		// 初始化注入
		container.initWired();
		
		Lol2 lol = container.getBean(Lol2.class);
		lol.work();
	}
	
	public static void iocNameTest(){
		container.registerBean("face", new FaceService2());
		container.registerBean(Lol3.class);
		// 初始化注入
		container.initWired();
		
		Lol3 lol = container.getBean(Lol3.class);
		lol.work();
	}
	
	public static void main(String[] args) {
		baseTest();
		//iocClassTest();
		//iocNameTest();
	}
	
}
```

输出结果：

**剑圣买了5毛钱特效，装逼成功!**

## GitHub

> [Github](https://github.com/machangchang/SimpleIOC)

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
