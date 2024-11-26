---
layout: post
title:  "Jekyll 搭建静态博客"
date:   2015-02-15 22:14:54
categories: jekyll
tags: mybatis
---

* content
{:toc}
一直都比较好奇Mybatis的参数名解析(@Value)  是怎么做的。虽然大致也能猜到是反射拿到注解信息。

今天从debug源码出发，开始分析是哪些类完成的命名解析。



# 1.@Value

mybatis支持使用`@Value` 来为Mapper方法的参数起别名:

```java
//fooMapper.java

List<TCtInfoManagerInfo> getTCtInfoManagerInfoByProjectSubId(@Value("id")String projectSubId,
                                                             @Value("foo")String foo,
                                                             @Value("foo1")String foo1);
```

并在xml中进行调用： 

```xml
    <select id="getTCtInfoManagerInfoByProjectSubId"
            resultType="com.bjdv.es.web.service.dto.TCtInfoManagerInfo">
		SELECT *
		FROM `project_sub_team` A LEFT JOIN `t_ct_info` B ON A.info_id = B.id 
        	  					  LEFT JOIN `sys_user` C ON C.id = B.manager
		WHERE A.sub_id = #{id} and A.foo = #{foo} and A.foo1 = {foo1}
	</select>
```



我们称这种为 参数名（ParamName）。







事实上每个参数都有一个默认名，我们可以这样使用: 

```java
//fooMapper.java

List<TCtInfoManagerInfo> getTCtInfoManagerInfoByProjectSubId(String projectSubId,
                                                             String foo,
                                                             String foo1);
```



```xml
    <select id="getTCtInfoManagerInfoByProjectSubId"
            resultType="com.bjdv.es.web.service.dto.TCtInfoManagerInfo">
		SELECT *
		FROM `project_sub_team` A LEFT JOIN `t_ct_info` B ON A.info_id = B.id 
        	  					  LEFT JOIN `sys_user` C ON C.id = B.manager
		WHERE A.sub_id = #{param1} and A.foo = #{param2} and A.foo1 = {param3}
	</select>
```



mybatis在解析命名的时候，为没有命名的参数，存入了默认命名。它取决于方法中参数的顺序，依次为：

param1 param2 param3...

并且，这个名称是冗余的名称，即使定义了`@Value` ，我们也可以使用。







命名解析规则是： 

- *如果参数使用* `@Param("<value>")` *修饰，那么参数的名称为*`<value>` 。 如果没有使用@param修饰，那么使用参数索引顺序。
- 对于特殊参数（RowBounds 或 ResultHandler ），不会算作param (也就是无法通过 param+index 来引用他们)



例如：

> aMethod(@Param("M") int a, @Param("N") int b)  则映射为  {{0, "M"}, {1, "N"}}
>
> aMethod(int a, int b)   则映射为  {{0, "0"}, {1, "1"}}
>
> aMethod(int a, RowBounds rb, int b)    则映射为  {{0, "0"}, {2, "1"}} 

