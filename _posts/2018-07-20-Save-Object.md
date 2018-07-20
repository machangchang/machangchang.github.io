---
layout: post
title: Java对象在Redis中的存取
categories: Java
description: Redis
keywords: Redis
---

使用Redis存储Java对象。

## 1. 被存储的对象

为了实现序列化的需求，该类实现了Serializable接口。

```java
public class Person implements Serializable {
	private int id;
	private String name;
 
	public Person(int id, String name) {
	  this.id = id;
	  this.name = name;
	}
 
	public int getId() {
	  return id;
	}
 
	public String getName() {
	  return name;
	} 
}
```

## 2. 序列化，反序列化工具类

```java
public class SerializeUtil {
	public static byte[] serialize(Object object) {
	ObjectOutputStream oos = null;
	ByteArrayOutputStream baos = null;
	try {
		//序列化
		baos = new ByteArrayOutputStream();
		oos = new ObjectOutputStream(baos);
		oos.writeObject(object);
		byte[] bytes = baos.toByteArray();
		return bytes;
	} catch (Exception e) {
		}
		return null;
	}
 
public static Object unserialize(byte[] bytes) {
	ByteArrayInputStream bais = null;
	try {
	//反序列化
	bais = new ByteArrayInputStream(bytes);
	ObjectInputStream ois = newObjectInputStream(bais);
	return ois.readObject();
	} catch (Exception e) {
	}
	return null;
	}
}
```

## 3. 写入对象到Redis中

```java
public void setObject() {
	Person person = new Person(100, "alan");
	jedis.set("person:100".getBytes(), SerializeUtil.serialize(person));
	person = new Person(101, "bruce");
	jedis.set("person:101".getBytes(), SerializeUtil.serialize(person));
```

## 4. 从Redis中获取对象

```java
public Person getObject(int id) {
	byte[] person = jedis.get(("person:" + id).getBytes());
	return (Person) SerializeUtil.unserialize(person);
}
```

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
