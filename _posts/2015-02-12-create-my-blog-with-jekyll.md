---
layout: post
title:  "Jekyll 搭建静态博客"
date:   2015-02-15 22:14:54
categories: jekyll
tags: mybatis
---

* content
{:toc}
一直都比较好奇Mybatis的参数名解析(@Param)  是怎么做的。虽然大致也能猜到是反射拿到注解信息。

今天从debug源码出发，开始分析是哪些类完成的命名解析。



# 1.@Param

mybatis支持使用@param 来为Mapper方法的参数起别名:

```java
//fooMapper.java

List<TCtInfoManagerInfo> getTCtInfoManagerInfoByProjectSubId(@Param("id")String projectSubId,
                                                             @Param("foo")String foo,
                                                             @Param("foo1")String foo1);
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

并且，这个名称是冗余的名称，即使定义了 param ，我们也可以使用。









# 2.参数名是如何解析的？

Mybatis通过`org.apache.ibatis.reflection.ParamNameResolver`完成参数解析的。



名称规则是
```java
如果参数使用 Param(value) 修饰，那么参数的名称为value 。
如果没有使用参数， 则使用参数的索引
对于特殊参数（RowBounds 或 ResultHandler ），不会算作param (也就是无法通过 param+index 来引用他们)
```









每一个MapperMethod ，都会创建一个参数名解析器（ParamNameResolver），在它的构造器中，完成了 

“ 参数在方法中的索引-> 参数名”的解析：

```java
public ParamNameResolver(Configuration config, Method method) {
  this.useActualParamName = config.isUseActualParamName();//默认true，使用参数真实的名称
  final Class<?>[] paramTypes = method.getParameterTypes();//每个参数的类型
  final Annotation[][] paramAnnotations = method.getParameterAnnotations();//每个参数的注解
  final SortedMap<Integer, String> map = new TreeMap<>();
  int paramCount = paramAnnotations.length;
  // 从每个参数中获得名称
  for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
    if (isSpecialParameter(paramTypes[paramIndex])) {//如果是特殊参数，则跳过
      // skip special parameters
      continue;
    }
    String name = null;
    for (Annotation annotation : paramAnnotations[paramIndex]) {//这里可以看到遍历了每个参数注解
      if (annotation instanceof Param) {
        hasParamAnnotation = true;
        name = ((Param) annotation).value();
        break;
      }
    }
    if (name == null) {
      // @Param was not specified.
      if (useActualParamName) {
        name = getActualParamName(method, paramIndex);
      }
      if (name == null) {
        // use the parameter index as the name ("0", "1", ...)
        // gcode issue #71
        name = String.valueOf(map.size());//参数没有声明名字，用map.size()代替
      }
    }
    map.put(paramIndex, name);
  }
  names = Collections.unmodifiableSortedMap(map);
}
```



随后，在获取参数名的时候，会使用 `GENERIC_NAME_PREFIX` 静态变量最为前缀，拼接索引+1 。

而`GENERIC_NAME_PREFIX` 的默认值为 `param` ，由此出现了默认名 `param1` `param2`

```java
//ParameterNameResolver.java

public Object getNamedParams(Object[] args) {
  final int paramCount = names.size();
  if (args == null || paramCount == 0) {
    return null;
  } else if (!hasParamAnnotation && paramCount == 1) {
      //没有 Param注解，且 只有1个参数的情况 ，返回参数中的第一个key。  
    Object value = args[names.firstKey()];//值总是为0
      //将参数返回为一个ParamMap(ibatis内部的Map，重写了get方法，当get不到key时抛出异常)  
      // 或 返回null
    return wrapToMapIfCollection(value, useActualParamName ? names.get(0) : null);
  } else {
      //如果是多参数，则执行这个分支
    final Map<String, Object> param = new ParamMap<>();
    int i = 0;
    for (Map.Entry<Integer, String> entry : names.entrySet()) {
        //从解析的names Map拷贝key 和value
      param.put(entry.getValue(), args[entry.getKey()]); 
        //添加通用的参数名称 （例如 param1, param2 ,param3），
      final String genericParamName = GENERIC_NAME_PREFIX + (i + 1);
      // 避免不会重写 `@Param("param1")`的情况
      if (!names.containsValue(genericParamName)) { //注意到是冗余的，只有出现冲突的时候，才不会放入
        param.put(genericParamName, args[entry.getKey()]);
      }
      i++;
    }
    return param;
  }
}
```



并且，这个方法将 List 、Array的参数都转换为了Map





