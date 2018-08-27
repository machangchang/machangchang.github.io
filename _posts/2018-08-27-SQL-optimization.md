---
layout: post
title: SQL优化技巧
categories: 数据库
description: SQL
keywords: Optimization
---

##SQL优化技巧

### 1. 避免在where子句中使用 is null 或 is not null 对字段进行判断

```sql
select id from table where name is null
```

在这个查询中，就算我们为 name 字段设置了索引，查询分析器也不会使用，因此查询效率底下。为了避免这样的查询，在数据库设计的时候，尽量将可能会出现 null 值的字段设置默认值，这里如果我们将 name 字段的默认值设置为''，那么我们就可以这样查询：

```java
select id from table where name = ''
```

### 2. 避免在 where 子句中使用 != 或 <> 操作符

如：`select name from table where id <> 0`

数据库在查询时，对 != 或 <> 操作符不会使用索引，而对于 < 、 <= 、 = 、 > 、 >= 、 BETWEEN AND，数据库才会使用索引。因此对于上面的查询，正确写法应该是：

```sql
select name from table where id < 0
 union all
 select name from table where id > 0
```

这里我们为什么没有使用 or 来链接 where 后的两个条件呢？这就是我们下面要说的第3个优化技巧。

### 3. 避免在 where 子句中使用 or来链接条件

如：

` select id from table where name = 'C++' or name = 'C#'`

这种情况，我们可以这样写：

```sql
select id from table where name = 'C++'
 union all
select id from table where name = 'C#'
```

### 4. 少用 in 或 not  in

虽然对于 in 的条件会使用索引，不会全表扫描，但是在某些特定的情况，使用其他方法也许效果更好。如：

```sql
select name from table where id in(1,2,3,4,5)
```

像这种连续的数值，我们可以使用 BETWEEN AND，如：

```sql
 select name from table where id between 1 and 5
```

### 5. 意 like 中通配符的使用

下面的语句会导致全表扫描，尽量少用。如：

```sql
select id from table where name like '%jayzai%'
```

或者：

```sql
select id from table where name like '%jayzai'
```

而下面的语句执行效率要快的多，因为它使用了索引：

```sql
 select id from table where name like 'jayzai%'
```

### 6. 避免在 where 子句中对字段进行表达式操作

如：

```sql
select name from table where id/2 = 100
```

正确的写法应该是：

```sql
select name from table where id = 100*2
```

### 7. 避免在 where 子句中对字段进行函数操作

如：

```sql
select id from table where substring(name,1,8) = 'jayzai'
```

或：

```sql
 select id from table where datediff(day,datefield,'2014-07-17') >= 0
```

这两条语句中都对字段进行了函数处理，这样就是的查询分析器放弃了索引的使用。正确的写法是这样的：

```sql
 select id from table where name like 'jayzai%'
```

或：

```sql
select id from table where datefield <= '2014-07-17'
```

### 8. 只要一行数据时使用 LIMIT 1

当你查询表的有些时候，你已经知道结果只会有一条结果，但因为你可能需要去fetch游标，或是你也许会去检查返回的记录数。

在这种情况下，加上 LIMIT 1 可以增加性能。这样一样，MySQL数据库引擎会在找到一条数据后停止搜索，而不是继续往后查少下一条符合记录的数据。

```sql
SELECT 1 FROM user WHERE country = 'China' LIMIT 1
```

其中 select 1 意思是查询出的值都为1，这样也可以提高查询效率。

### 9. 避免 SELECT *

从数据库里读出越多的数据，那么查询就会变得越慢。并且，如果你的数据库服务器和WEB服务器是两台独立的服务器的话，这还会增加网络传输的负载。

应该养成一个需要什么就取什么的好的习惯。

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
