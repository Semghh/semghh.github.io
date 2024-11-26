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




