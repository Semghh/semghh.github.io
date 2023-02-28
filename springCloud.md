# 1.认识微服务



## 1.1. 传统单体架构

![image-20220312144836703](springCloud.assets/image-20220312144836703.png)

```
适合企业内部使用小型项目。不适合模块众多的大型项目。
模块之间耦合度较高。
```

## 1.2. 分布式架构



![image-20220312144953233](springCloud.assets/image-20220312144953233.png)





![image-20220312145041819](springCloud.assets/image-20220312145041819.png)





分布式带来的思考:

![image-20220312145208684](springCloud.assets/image-20220312145208684.png)

```
远程调用：不在同一台机器上调用代码。
```



## 1.3 微服务

![image-20220312145432979](springCloud.assets/image-20220312145432979.png)



### 1.3.1 微服务特征

![image-20220312150024533](springCloud.assets/image-20220312150024533.png)



```
这些的思想都是为了完成，高内聚，低耦合。降低服务之间的影响。
```





![image-20220312150157126](springCloud.assets/image-20220312150157126.png)







![image-20220312150439394](springCloud.assets/image-20220312150439394.png)





### 1.3.2 微服务对比





![image-20220312151043277](springCloud.assets/image-20220312151043277.png)



### 1.3.3 企业可能会使用模式



![image-20220312151516623](springCloud.assets/image-20220312151516623.png)



## 1.4 springCloud



![image-20220312151746374](springCloud.assets/image-20220312151746374.png)





### 1.4.1 远程过程调用

使用 RestTemplate 对象发起调用。从各个服务中发https请求。使用返回来的json对象。

反序列化以后，组合结果，返回给用户。



![image-20220312154411526](springCloud.assets/image-20220312154411526.png)







# 2. Eureka

```
Eureka是SpringCloud的注册中心组件。
```





![image-20220312154721913](springCloud.assets/image-20220312154721913.png)





## 2.1 Eureka基本概念



### 2.1.1  提供者/消费者



![image-20220312154600974](springCloud.assets/image-20220312154600974.png) 





### 2.1.2 远程调用的问题



在1.4.1中，url是写死在代码中的。如果是集群服务，服务的地址有多个，该怎么办？所以，url也应该是动态请求的。所以需要“注册中心”。



![image-20220312155223521](springCloud.assets/image-20220312155223521.png)



```
对于Eureka来说，自己是Eureka-server 服务端。
所有的其他服务，都称之为Eureka-client 客户端。
```





服务注册流程：

```
1.各种服务,在部署的最开始，都要向Eureka-server注册自己。此时Eureka-server会记录各种服务的url。
2.当client想要调起其他服务，都会向server请求目标服务的url。此时server会动态的返回目标服务的url。此时,可以使用负载均衡。
```

Eureka-Server返回的服务url会不会是已经宕机的？

```
不会，所有服务每隔30s都要向server发起一次心跳续约。解决了宕机问题。
```



### 2.1.3 Eureka的作用



![image-20220312155756347](springCloud.assets/image-20220312155756347.png)





### 2.1.4 总结

![image-20220312155812684](springCloud.assets/image-20220312155812684.png)







## 2.2 搭建Eureka



### 2.2.1  Eureka-server

![image-20220312160109230](springCloud.assets/image-20220312160109230.png)



```
注意，引入的依赖是  -eureke-server
```





### 2.2.2 Eureka-client





![image-20220312165600789](springCloud.assets/image-20220312165600789.png)



```
先引入Eureka的jar包。 spring-cloud-starter-netfix-eureka-client

将自己的服务名称修改：  spring.application.name: xxxService

再配置Eureka.client.service-url.defaultZone
```





### 2.2.3 为服务配置集群



![image-20220312165833664](springCloud.assets/image-20220312165833664.png)



### 2.2.4 在Controller中修改服务url

```
之前的RestTemplate中url是写死的。
现在将从Eureka-server 根据服务名动态获取url。并且,可以使用负载均衡。


对RestTemplate实例标注@LoadBalanced实现负载均衡。
```



![image-20220312170236505](springCloud.assets/image-20220312170236505.png)





### 2.2.5 总结



![image-20220312170548525](springCloud.assets/image-20220312170548525.png)





# 3. Ribbon

Ribbon 负责 负载均衡。



## 3.1 Ribbon 工作流程

整体流程： 

![image-20220312170751755](springCloud.assets/image-20220312170751755.png)



细化：

![image-20220312171851633](springCloud.assets/image-20220312171851633.png)



## 3.2 负载均衡策略



```
负载均衡策略接口： IRule 
通过实现IRule 可以自定义负载均衡策略。比如为一些接口增加权重。
```





![image-20220312172010192](springCloud.assets/image-20220312172010192.png)



常见的负载均衡策略。



![image-20220312172218606](springCloud.assets/image-20220312172218606.png)



## 3.3 修改负载均衡策略

```
通过往容器中注入实现了IRule接口的实现类实例。
```





![image-20220312172446640](springCloud.assets/image-20220312172446640.png)





```
可以通过，针对某一个微服务的负载均衡。

```





![image-20220312172839058](springCloud.assets/image-20220312172839058.png)





## 3.4  饥饿加载



![image-20220312173303327](springCloud.assets/image-20220312173303327.png)





![image-20220313083722369](springCloud.assets/image-20220313083722369.png)









# 4. Nacos

 Nacos 也是注册中心.



![image-20220313084115517](springCloud.assets/image-20220313084115517.png)





##  4.1 安装Nacos

nacos配置文件在 Nacos 根目录下 conf/application.properties

![image-20220313095045179](springCloud.assets/image-20220313095045179.png)

启动在bin/startup.cmd





单机模式下启动：

```
starup.cmd -m standalone
```





Console网址：帐号密码都是 nacos

![image-20220313095406375](springCloud.assets/image-20220313095406375.png)





## 4.2  服务注册/发现



```
首先引入Nacos依赖。
```





![image-20220313095700699](springCloud.assets/image-20220313095700699.png)

```
引入Nacos服务发现的依赖
```





修改父工程配置文件

![image-20220313095948370](springCloud.assets/image-20220313095948370.png)



```
首先需要配置Nacos注册中心的服务地址。任意一个微服务，都需要把自己注册到微服务中。以便于其他服务在调用的时候，从Nacos注册中心能拉取到自己。
```



```
spring.cloud.nacos.server-addr=127.0.0.1:8848
```













### 4.2.1 服务分级存储模型



```
同一类服务，可能有多个集群，而这些集群分属于不同的地区。
```







![image-20220313100520545](springCloud.assets/image-20220313100520545.png)



```
Nacos采取分级存储模型，是为了解决集群调用问题。

Nacos称一个Zone 为一个集群。同一个Zone内的服务调用服务,会优先调用同一个Zone的目标服务。
```

![image-20220313100716727](springCloud.assets/image-20220313100716727.png)

```
显然优先调用本地集群可以获得更高的性能，更好的服务。当本地集群挂了以后，跨Zone调用又能保证服务的高可用。
```







### 4.2.2 配置服务集群属性



![image-20220313100940874](springCloud.assets/image-20220313100940874.png)







## 4.3 Nacos负载均衡



![image-20220313101817186](springCloud.assets/image-20220313101817186.png)

```
调用者:orderService 也添加Cluster-name属性。
修改负载均衡规则。
```

![image-20220313101956074](springCloud.assets/image-20220313101956074.png)



```
Nacos策略: 优先访问同Cluster-name的目标服务且是随机访问。当同集群下没有可用的目标服务,再尝试访问其他集群的目标服务。
```





### 4.3.1 根据权重配置负载均衡



![image-20220313102834791](springCloud.assets/image-20220313102834791.png)

​	

![image-20220313102854594](springCloud.assets/image-20220313102854594.png)





## 4.4 NameSpace 环境隔离





![image-20220313113340266](springCloud.assets/image-20220313113340266.png)





![image-20220313113746029](springCloud.assets/image-20220313113746029.png)



![image-20220313114032992](springCloud.assets/image-20220313114032992.png)

## 4.5 临时实例/非临时实例



![image-20220313120528532](springCloud.assets/image-20220313120528532.png)

```
Nacos将服务实例分为 临时实例/非临时实例

临时实例使用心跳包续约。非临时实例由Nacos主动询问服务是否宕机。

当临时实例宕机以后,服务列表中会直接剔除该实例。非临时实例仍在列表中，但状态会显示为宕机


并且当服务列表变更以后，Nacos服务器会主动推送变更信息到各个消费者。
```



![image-20220313121001989](springCloud.assets/image-20220313121001989.png)





## 4.6 Nacos配置管理

![image-20220313122959595](springCloud.assets/image-20220313122959595.png)



### 4.6.1 统一配置管理

```
在大的机房中，可能有很多台机器。当配置更改时，如果需要手动配置每一台机器这显然很麻烦。
所以需要统一配置管理。并且希望配置热更新，服务不用重启。
```



![image-20220313143315079](springCloud.assets/image-20220313143315079.png)

```
为不同服务的不同配置文件起一个唯一ID
```



#### 4.6.1.1 统一配置流程



传统模式下:

![image-20220313143825184](springCloud.assets/image-20220313143825184.png)

```
传统单机项目: 项目启动，读取application.yaml中的配置。 根据配置里的信息，创建Spring容器(加载bean的同时也会从配置中读取数据)
```





使用统一配置以后：

![image-20220313143850223](springCloud.assets/image-20220313143850223.png)

```
项目启动，先根据一个”启动项目配置文件”，找到配置中心，拉取相关配置信息。再读取本地配置文件，进行合并。然后创建spring容器。

“启动项目配置文件” 就是 bootstrap.yaml
```





![image-20220313144046465](springCloud.assets/image-20220313144046465.png)

```
在spring中,比application配置先加载的配置信息。 bootstrap.yaml
```





#### 4.6.1.2 引入Nacos统一配置依赖



```
在多台集群的场景下，手动为每一台机器配置变量是非常麻烦的操作。引入了配置中心以后，可以实现1处修改，多处变化的优点。同时，还支持动态改变变量的值。
```



使用Nacos统一配置，需要额外引入依赖。

![image-20220313144157277](springCloud.assets/image-20220313144157277.png)











在bootstrap.yaml中配置，同时删除application.yaml中重复的部分。

![image-20220313144904516](springCloud.assets/image-20220313144904516.png)









### 4.6.2 配置热更新



```
使用注册中心，可以动态的刷新@Value 和@ConfigurationProperties 变量的值。
前提是，这个Controller被标注了@RefreshScope 注解
```





#### 4.6.2.1 刷新@Value

![image-20220313145235409](springCloud.assets/image-20220313145235409.png)



#### 4.6.2.2 刷新@ConfigurationProperties

![image-20220313145747081](springCloud.assets/image-20220313145747081.png)









### 4.6.3 多环境配置共享

![image-20220313150203299](springCloud.assets/image-20220313150203299.png)







#### 4.6.3.1 优先级



![image-20220313151013986](springCloud.assets/image-20220313151013986.png)



### 4.6.4 Nacos集群

![image-20220313151334628](springCloud.assets/image-20220313151334628.png)

#### 4.6.4.1 搭建方法





![image-20220313160938742](springCloud.assets/image-20220313160938742.png)

```
将cluster.conf.example 改为 cluster.conf。 并修改集群ip地址。

以单点为例: 
127.0.0.1:8845
127.0.0.1:8846
127.0.0.1:8847
```



![image-20220313161249987](springCloud.assets/image-20220313161249987.png)





```
为每一个nacos实例配置
修改端口号
修改数据库平台
数据库数量
数据库url
username
password
```

![image-20220313161432917](springCloud.assets/image-20220313161432917.png)















以mysql数据库为例。创建数据库 “nacos” 。 表信息如下：

```sql
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) DEFAULT NULL,
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) DEFAULT NULL,
  `c_use` varchar(64) DEFAULT NULL,
  `effect` varchar(64) DEFAULT NULL,
  `type` varchar(64) DEFAULT NULL,
  `c_schema` text,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
  `content` longtext NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) NOT NULL,
  `group_id` varchar(128) NOT NULL,
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL,
  `md5` varchar(32) DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `src_user` text,
  `src_ip` varchar(50) DEFAULT NULL,
  `op_type` char(10) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';


CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

CREATE TABLE `users` (
	`username` varchar(50) NOT NULL PRIMARY KEY,
	`password` varchar(500) NOT NULL,
	`enabled` boolean NOT NULL
);

CREATE TABLE `roles` (
	`username` varchar(50) NOT NULL,
	`role` varchar(50) NOT NULL,
	UNIQUE INDEX `idx_user_role` (`username` ASC, `role` ASC) USING BTREE
);

CREATE TABLE `permissions` (
    `role` varchar(50) NOT NULL,
    `resource` varchar(255) NOT NULL,
    `action` varchar(8) NOT NULL,
    UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
);

INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);

INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```





启动所有nacos实例







使用Nginx做集群映射。

![image-20220313161554832](springCloud.assets/image-20220313161554832.png)



```
springcloud项目修改。集群映射地址。 例如此时nginx使用80端口作为 nacos-cluster upstream的监听端口。
则改为 spring.cloud.nacos.server-addr: localhost:80
```



![image-20220313161626057](springCloud.assets/image-20220313161626057.png)













# 5.  Feign

Http客户端，用于代替RestTemplate





为什么Feign要代替Restful  因为URL请求风格的代码，在复杂参数情况下，可读性比较差，不够优雅。![image-20220313162212429](springCloud.assets/image-20220313162212429.png)

## 5.1 引入Feign



![image-20220313162736212](springCloud.assets/image-20220313162736212.png)



## 5.2 Feign的使用



### 5.2.1 引入依赖



![image-20220313183950315](springCloud.assets/image-20220313183950315.png)



​	











### 5.2.2 完整流程



#### 引入依赖



#### 定义接口

![image-20220313184703268](springCloud.assets/image-20220313184703268.png)



```
此时发送的Http请求是 http://userService/user/{id}   将方法参数id 当作路径变量拼接到url中。发送请求，并把返回的json 转化为User对象。

此时需要搭配注册中心。注册中心的Http拦截器将userService替换为确定的url
```



在@FeignClient 注解上必须标注服务的名称。 `FeignClient("userservice")`

```
这意味着,Feign需要搭配注册中心使用。动态的从注册中心拉取服务对应的ip地址
```



远程过程调用并不需要实现方法。所以@FeignClient标注在接口即可。



接口内的方法与调用的远程接口保持完全一致即可;

```
如果对方是Get，则如同开发一样，使用@GetMapping即可。Post则@PostMapping.
同样支持路径变量
```



![image-20220604164059212](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220604164059212.png)

















#### 在配置类上开启Feign 。 @EnableFeignClients

![image-20220313184330900](springCloud.assets/image-20220313184330900.png)





#### 在Service层注入



![image-20220313184956152](springCloud.assets/image-20220313184956152.png)





#### 5.2.2.1 完整流程实践



使用Eureka注册中心、Feign作为RPC调用 

引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

Eureka-Server 引入Server依赖，并且配置类上开启EurekaServer：

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)//不配置加载数据源
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(EurekaApplication.class);
    }
}
```

完成Eureka配置。  [详情参考2.2.1](#2.2.1  Eureka-server)



定义Feign的Client接口。

```java
@FeignClient("userService")     //使用服务名代替地址
public interface UserClient {
    @GetMapping("/user/{id}")   //类Restful风格
    User getUserById(@PathVariable("id") Long id);//表示是一个路径变量。在发Http请求的时候会拼接到url上
}
```

实现UserServiceImpl:

​		注入userClient

```java
    @Resource
    private UserClient userClient;
```

​		实现接口方法:

```java
@Override
public Order getOrderByIdFeign(Long id) {
    Order orderById = orderMapper.getOrderById(id);
    User userById = userClient.getUserById(orderById.getUserId());
    orderById.setUser(userById);
    return orderById;
}
```



















## 5.3 Feign的自定义配置



![image-20220313185228309](springCloud.assets/image-20220313185228309.png)





### 5.3.1 配置Feign的日志



```
在application.yaml中配置
```



![image-20220313185532061](springCloud.assets/image-20220313185532061.png)







```
使用Bean配置、注解配置
```

![image-20220313185820350](springCloud.assets/image-20220313185820350.png)



## 5.4 Feign性能优化



![image-20220313190133820](springCloud.assets/image-20220313190133820.png)

```
URLConnection 性能不够好，不支持连接池
```





![image-20220313190307373](springCloud.assets/image-20220313190307373.png)

### 5.4.1  优化连接池



![image-20220313190359050](springCloud.assets/image-20220313190359050.png)





## 5.5 Feign 最佳实践



### 5.5.1 继承

![image-20220313193047369](springCloud.assets/image-20220313193047369.png)



![image-20220313193113723](springCloud.assets/image-20220313193113723.png)

```
Spring官方并不推荐这种使用统一接口标准的行为。因为这样会造成了一种紧耦合。并且@PathVariable不会生效。必须在实现方法上再次标注。
```



![image-20220313193343339](springCloud.assets/image-20220313193343339.png)







### 5.5.2 抽取



```
鉴于前文继承带来的问题,各自的微服务应当定义自己的 Client。但是如果某一种Client被其他微服务使用频繁。
有数十种甚至更多的微服务都需要调用这个client服务。那么应该由userService服务提供者抽象出一个Model,供大家调用。
```



初始：

![image-20220313193519881](springCloud.assets/image-20220313193519881.png)





抽取：

![image-20220313193844090](springCloud.assets/image-20220313193844090.png)

抽取出一个module，其他微服务引入jar包 调用api即可。



#### 5.5.2.1 抽取实战



![image-20220315135531667](springCloud.assets/image-20220315135531667.png)

![image-20220315140806612](springCloud.assets/image-20220315140806612.png)

​	



```
在Order-Serivce的配置类上。
@EnableFeignClients注解属性 clients，引入了 feign-api的 UserClient.class

否则。在OrderApplication的Spring容器中，没有这个对象。
```



![image-20220315143158854](springCloud.assets/image-20220315143158854.png)











## 5.6 Feign使用的注意事项



### 5.6.1 @RequestParam

在SpringMVC中，@RequestParam()用于标注参数，表示这个参数是一个GET参数。Feign也使用了这个参数。



```java
    @GetMapping("/sms/send")
    R smsSendCode(@RequestParam("phone") String phone);
```



在Feign中，会把被@RequestParam标注的参数，解析为GET类型的请求参数。





#### 5.6.1.1 GET方法会被解析为POST方法

如果不使用@RequestParam标注参数，那么会出现一个致命的问题：

```
GET方法会被解析为POST方法
```



原因是，如果没有任何注解标注的 Method Param,会被默认解析为 RequestBody中的JSON。而Feign会强制把包含了RequestBody的请求改为 POST请求。



```
所以，如果是简单类型的GET参数，必须标注@RequestParam

如果是复杂类型的GET参数必须标注  @SpringQueryMap
```





参考博客

https://blog.csdn.net/zcswl7961/article/details/106328174











### 5.6.2 @SpringQueryMap

使用@QueryMap 表明这是一个复杂的Vo将被解析成 Get请求的Param

```java
@GetMapping("/sms/check")
R smsCheckCode(@QueryMap SmsCheckVo vo);
```



使用原因如5.6.1.1



































# 6.Gateway 网关

![image-20220315143409516](springCloud.assets/image-20220315143409516.png)



## 6.1 为什么需要网关



```
一些微服务需要暴露到公网，供用户调用。
而更多的微服务,作为支撑服务或者敏感业务不能暴露到公网。只能使用内网调用。此时就需要网关	
```





![image-20220315143601305](springCloud.assets/image-20220315143601305.png)





![image-20220315143830685](springCloud.assets/image-20220315143830685.png)





## 6.2 搭建网关



### 	6.2.1  引入依赖

![image-20220315143918860](springCloud.assets/image-20220315143918860.png)



### 6.2.2 配置gateway



![image-20220315162327268](springCloud.assets/image-20220315162327268.png)



```
gateway本身也是一个微服务，所以也需要注册进入Nacos

spring.cloud.nacos.server-addr = localhost:8848 #默认端口
```







```
spring.cloud.gateway.routes

路由的三部分：
路由的id、
路由的url地址、
路由的规则
```



### 6.2.3 Gateway流程

![image-20220315170047299](springCloud.assets/image-20220315170047299.png)





## 6.3 路由断言



![image-20220315170205203](springCloud.assets/image-20220315170205203.png)



断言工厂读取用户配置的断言，并解析成判断规则。

![image-20220315170230111](springCloud.assets/image-20220315170230111.png)





### 6.3.1 spring提供的断言工厂



![image-20220315170336940](springCloud.assets/image-20220315170336940.png)



### 6.3.2 断言示例



#### 6.3.2.1 Path

```
匹配路径符合指定规则。
```



```yaml
spring: 
	cloud: 
		gateway: 
        	routes:
            	- id: coutent2_route
                  uri: lb://content2    //  lb://<服务名称application.name> 负载均衡
                  predicates:
                    - Path= /content2/**
```

```
以上这个Gateway的routes配置，使用了Path断言。
匹配路径等于 /content2下的任意路径，将这个路径请求，原封不动的发送到  content2服务的地址下。(原封不动的意思是指/content2/ 也会被一同请求转发)
```



如果不想请求一并转发呢，可以使用Gateway的请求重写。

```yaml
#请求重写就是Filter的功能了
# content2服务能正确处理  /content1-2.0/**  的请求。
#前端发的路径是 /content2/content1-2.0/**
#现在Gateway网关想要拦截 /content2/**的请求 并重写为 /content1-2.0/**

spring: 
	cloud: 
		gateway: 
        	routes:
            	- id: coutent2_route
                  uri: lb://content2    //  lb://<服务名称application.name> 负载均衡
                  predicates:
                    - Path= /content2/**
                  filters:
                    - RewritePath=/content2/(?<uri>.*),/$\{uri}
```

```
表示 /content2/后面的全部路径uri，都会被重写为  /uri
```



![image-20220514161052732](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220514161052732.png)











官方示例：

![image-20220514151458286](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220514151458286.png)

![image-20220514151914157](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220514151914157.png)

```
这个路由断言可以匹配   /red/1  /red/1/ 
如果  matchTrailingSlash(默认是true 表示匹配最后一个/) 如果是false表示不匹配
```







![image-20220514151603469](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220514151603469.png)


```
Path断言可以截取URI模板的变量(示例中的{segment})同时，会把这些变量封装进一个Map中(ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE)。
同过ServerWebExchange.getAttributes()可以调用。
这些键值可以被后续的 GatewayFilter使用
```

一个使用示例如下:



![image-20220514152936629](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220514152936629.png)









## 6.4 路由过滤器



![image-20220315170920476](springCloud.assets/image-20220315170920476.png)



网关过滤器的工作示意图



![image-20220315171231636](springCloud.assets/image-20220315171231636.png)

### 6.4.1   spring提供的过滤器

spring官方文档地址

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories





![image-20220315171316279](springCloud.assets/image-20220315171316279.png)





![image-20220315172324982](springCloud.assets/image-20220315172324982.png)



```
添加请求头
添加请求参数
添加响应头	
```





![image-20220315173006398](springCloud.assets/image-20220315173006398.png)







#### 6.4.1.1   修改URL前缀

![image-20220315173106954](springCloud.assets/image-20220315173106954.png)

```
/hello 将被发送到  /mypath/hello
```



#### 6.4.1.2  请求限流



![image-20220315173400801](springCloud.assets/image-20220315173400801.png)







redis 限流

![image-20220315173702831](springCloud.assets/image-20220315173702831.png)







#### 6.4.1.3 重定向

![image-20220315173910418](springCloud.assets/image-20220315173910418.png)





#### 6.4.1.4  自定义默认过滤器



自定义一个默认过滤器。其他的路由可以通过 “spring.cloud.gateway.default-filters” 使用它



![image-20220315174106583](springCloud.assets/image-20220315174106583.png)

```
注意，默认过滤器和 routes 是同一级的参数
```





![image-20220315175729627](springCloud.assets/image-20220315175729627.png)







### 6.4.2  自定义过滤器 GlobalFilter



![image-20220315180300948](springCloud.assets/image-20220315180300948.png)





![image-20220315182741968](springCloud.assets/image-20220315182741968.png)



@Order 注解 和实现  Orderd 接口一样。 数值越小优先级越高。









### 6.4.3 过滤器执行顺序





 	![image-20220316091230157](springCloud.assets/image-20220316091230157.png)





## 6.5 处理跨域问题



![image-20220316091643299](springCloud.assets/image-20220316091643299.png)



![image-20220514162507013](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220514162507013.png)











# 7. MQ



## 7.1 引入消息队列

### 7.1.1 同步调用的问题



![image-20220316092135089](springCloud.assets/image-20220316092135089.png)

```
优点：
简单直观，易于编写。整个流程的控制，比较方便。
总的占用资源较少。
```



```
缺点：
耦合度高
性能下降：服务调用者需要等待服务提供者响应。
资源浪费： 等待响应的过程中,不能释放请求占用的资源
级联失败：服务提供者出现问题，所有调用方都会跟着出现问题。
```



### 7.1.2 异步调用



异步调用常见的一种模式：事件驱动。



```
优点：
解耦合 、解决级联失败问题。
提升吞吐量 
削峰填谷： 流量高峰期时候入MQ,后续的服务按照合理的速度处理消息。
执行效率高： 不需要服务调用者等待。
```







### 7.1.3  市面的MQ对比



![image-20220316093417548](springCloud.assets/image-20220316093417548.png)



```
kafka 可能会出现数据丢失，但吞吐量很高。所以常用于一些数据量大，安全性要求不高的服务。比如日志
```



```
RabbitMQ    RocketMQ  安全性较高，吞吐量不算太差。用于服务之间的通信。
```









## 7.2 在Docker中启动MQ



在docker中pull 带有Management版本的rabbitmq

![image-20220316134223308](springCloud.assets/image-20220316134223308.png)



```
部署MQ 
-e 表示配置参数。也就是为MQ启动时带参数

下面配置的rabbitMQ的用户名和密码
```





![image-20220316133402965](springCloud.assets/image-20220316133402965.png)

启动成功后，访问rabbitMQ的管理界面。

![image-20220316134401583](springCloud.assets/image-20220316134401583.png)



#### 7.4.1  RabbitMQ 结构示意图

![image-20220316135150040](springCloud.assets/image-20220316135150040.png)

![image-20220316135214125](springCloud.assets/image-20220316135214125.png)



## 7.5  常见的消息模型



![image-20220316152359496](springCloud.assets/image-20220316152359496.png)





### 7.5.1 hello-world 模型



```
hello-world 模型使用的就是 basicQueue。
即 1个Queue 1个Publisher 1个consumer
```







![image-20220316152500655](springCloud.assets/image-20220316152500655.png)



.







## 7.6 SpringAMQP



![image-20220316164822744](springCloud.assets/image-20220316164822744.png)



### 7.6.1 AMQP



![image-20220316164946284](springCloud.assets/image-20220316164946284.png)

```
高级消息队列协议。
```

### 7.6.2 使用RabbitTemplate



#### 7.6.2.1 引入依赖

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```



#### 7.6.2.2 配置yaml

![image-20220316165841926](springCloud.assets/image-20220316165841926.png)







#### 7.6.2.3 编写订阅逻辑

也就是编写消费者逻辑



![image-20220316182857674](springCloud.assets/image-20220316182857674.png)





### 7.6.3   续7.5消息模型



#### 7.6.3.1 工作队列 workQueue

![image-20220317082228138](springCloud.assets/image-20220317082228138.png)

```
1个Publicher 1个Queue  2个Consumer 
```



![image-20220317082258637](springCloud.assets/image-20220317082258637.png)



```
按照上述逻辑,安排两个消费者，消费消息。打印信息如下:
发现work1 总是处理奇数、work2总是处理偶数，这意味着2者总是分配了平均的消息。

并没有按照两个消费者实际能力分配消息，性能优秀的服务器没有得到充分发挥。
```



![image-20220317092507607](springCloud.assets/image-20220317092507607.png)



原因是Rabbit 消息预取机制。 

（猜测，每一个监听workQueue的Listener都有一个自己的预取队列，当workQueue收到消息以后，分发到各个Listener的预取队列，就相当于被这个Listener消费了，在未来的某个时间点，这个Listener总会去消费这条消息)



```
在配置项中可以修改预取队列的长度，如果设置为1、则表示上一条消息实际消费完了，才去获取下一条消息。可以反映各个机器的实际能力。
```



![image-20220317093436095](springCloud.assets/image-20220317093436095.png)







```
修改配置项以后，重新尝试获取消息。达到按照机器实际处理能力的效果。
```

![image-20220317093710483](springCloud.assets/image-20220317093710483.png)









#### 7.6.3.2 * 发布订阅



![image-20220317093908430](springCloud.assets/image-20220317093908430.png)



![image-20220317094259243](springCloud.assets/image-20220317094259243.png)



##### 7.6.3.2.1 交换机常见类型：



![image-20220317094227104](springCloud.assets/image-20220317094227104.png)



###### 7.6.3.2.1.1 Fanout 广播





![image-20220317094402645](springCloud.assets/image-20220317094402645.png)





Fanout Exchange 的工作模式图：



![image-20220317094453060](springCloud.assets/image-20220317094453060.png)



```
也就是说,发布者并不需要bind队列，而是bind交换机即可。让交换机完成队列的转交工作。
```











###### 7.6.3.2.1.2 广播模式 实战



![image-20220317094623994](springCloud.assets/image-20220317094623994.png)

```
SpringAMQP 帮我们化简了 注册交换机，声明队列，绑定队列的过程。

步骤：
1. 在Consumer服务中，添加一个配置类@Configuration。在其中注入Bean。

2. 编写监听服务。 @RabbitListener(queues = {})

3. 向交换机发送消息。RabbitTemplate.convertAndSend()
```



```
注意在发送 Fanout 广播的时候， routeKey 必须填"" ，不能直接省略。
```











第一步：

<img src="springCloud.assets/image-20220317103751463.png" alt="image-20220317103751463" style="zoom:50%;" />

交换机的继承图如上：  只需要new一个FanoutExchange，并注入Bean。SpringAMQP就自动帮我们完成了向RabbitMQ中声明交换机的过程。

```java
    @Bean("fanoutExchange")
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("crazyhh.fanout"); //创建一个交换机，指明交换机名字
    }
```

同样的：只需new一个Queue ,SpringAMQP 就自动完成声明队列

```java
    @Bean("myQueue1")
    public Queue fanoutQueue(){
        return new Queue("fanout.myQueue1");
    }// org.springframework.amqp.core.Queue;
```

同样的：  交换机和队列绑定也通过注入Bean的方式完成。

```java
    @Bean("bindingMyQueue1")
    public Binding bindingMyQueue1(@Qualifier(value = "myQueue1") Queue myQueue,
                                  @Qualifier(value = "fanoutExchange") FanoutExchange fanoutExchange){
        return BindingBuilder.bind(myQueue).to(fanoutExchange);
    }
```



同样的方式，声明队列 myQueue2 ，绑定到交换机中。

```java
@Bean("myQueue2")
    public Queue fanoutQueue2(){
        return new Queue("fanout.myQueue2");
    }

    @Bean("bindingMyQueue2")
    public Binding bindingMyQueue2(@Qualifier(value = "myQueue2") Queue myQueue,
                                  @Qualifier(value = "fanoutExchange") FanoutExchange fanoutExchange){
        return BindingBuilder.bind(myQueue).to(fanoutExchange);
    }
```









第二步： 编写Listener逻辑



```java
@Component
public class FanoutConsumer {

    @RabbitListener(queues = {"fanout.myQueue1"})
    public void fanoutConsumer1(String msg){
        System.out.println("监听[fanout.myQueue1]收到消息 : " + msg);
    }//监听队列

    @RabbitListener(queues = {"fanout.myQueue2"})
    public void fanoutConsumer2(String msg){
        System.out.println("监听[fanout.myQueue2]收到消息 : " + msg);
    }//监听队列
}
```







第三步：发布消息

```java
@Test
public void testFanout(){
    String exchange = "crazyhh.fanout";//向交换机发送消息
    String msg = "hell,everyone";//消息本体
    rabbitTemplate.convertAndSend(exchange,"",msg); //暂时没有用到routeKey
    //仍然使用rabbitTemplate发送
}
```

结果：

<img src="springCloud.assets/image-20220317112444619.png" alt="image-20220317112444619" style="zoom: 50%;" />



###### 7.6.3.2.1.3  总结Fanout

![image-20220317112826234](springCloud.assets/image-20220317112826234.png)











###### 7.6.3.2.1.4  direct 路由模式



![image-20220317112947717](springCloud.assets/image-20220317112947717.png)





![image-20220317113151360](springCloud.assets/image-20220317113151360.png)

```
Publisher发送消息到Exchanger以后,通过匹配 routeKey 和 bindingKey 对暗号。
如果匹配，则发送到对应的Queue。

注意： Queue和Bindkey 是多对多的关系。1个Queue可以绑定多个key,1个key也可以绑定多个Queue
```



![image-20220317113427102](springCloud.assets/image-20220317113427102.png)



```
比Fanout更灵活，但也更复杂。在路由的时候一定会带来比Fanout更多的性能损失。
```





###### 7.6.3.2.1.5  direct 路由 实战



![image-20220317113818936](springCloud.assets/image-20220317113818936.png)



```
如果觉得使用Bean的方式太复杂。可以使用@RabbitListener 配置Exchange,Queue,RoutingKey。

但使用注解配置复杂的内容，可读性一般不高。
```



下面给出了一种使用注解配置的方法：

![image-20220317120912977](springCloud.assets/image-20220317120912977.png)



```
注意，这种方式，同时完成了 exchange 、 bindingKey ,QueueName ,Listener 的定义。
源码如下:
```



```java
@Component
public class DirectListener {
    
    private static final String exchangeName =  "crazyhh.direct";
    private static final String queueName1 = "direct.queue1";

    @RabbitListener(bindings = @QueueBinding(value = @Queue(name = queueName1),
                                            exchange = @Exchange(name = exchangeName ,type = ExchangeTypes.DIRECT),
                                            key = {"red","blue"}))
    public void redBlueDirectListener(String msg){
        System.out.println("监听[exchange :"+exchangeName+"].[queue : "+queueName1+"].[routeKey : red,blue] 收到消息 "+
                msg);
    }


    private static final String queueName2 = "direct.queue2";

    @RabbitListener(bindings = @QueueBinding(   exchange = @Exchange(name = exchangeName,type = ExchangeTypes.DIRECT),
                                                value = @Queue(name = queueName2),
                                                key = {"red","yellow"}))
    public void redYellowDirectListener(String msg){
        System.out.println("监听[exchange :"+exchangeName+"].[queue :"+queueName2+"].[routeKey : red,blue] 收到消息 "+
                msg);
    }
}
```



编写publisher 代码：

![image-20220317150119536](springCloud.assets/image-20220317150119536.png)



结果:

![image-20220317150135351](springCloud.assets/image-20220317150135351.png)



```
当routeKey 为red 的时候, queue1 和queue2 都会收到消息。 //4次
当routeKey 为blue 1次
当routeKey 为yellow 1次
```





###### 7.6.3.2.1.6  direct 路由实战2





现在使用Bean来完成(类似的)配置：

```java
@Configuration
public class directBinding {

    //声明Exchange
    @Bean("directExchange")
    public DirectExchange directExchange(){
        return new DirectExchange("semghh.direct");
    }

    //声明Queue
    @Bean("RPCqueue")
    public Queue FeignRestfulQueue(){
        return new Queue("direct.feignRestful");
    }

    @Bean("RegisterAndDiscoveryQueue")
    public Queue nacosQueue(){
        return new Queue("direct.nacos");
    }

    //绑定
    @Bean
    public Binding nacosBinding(@Qualifier("directExchange") DirectExchange exchange,
                                @Qualifier("RegisterAndDiscoveryQueue") Queue nacosQueue){
        return BindingBuilder.bind(nacosQueue).to(exchange).with("nacos");
    }



    // 由于一个bindingKey 对应着一个Binding 。当队列需要绑定多个 routeKey的时候,需要注入多个Binding
    @Bean
    public Binding RpCBindingFeign(@Qualifier("directExchange") DirectExchange exchange,
                              @Qualifier("RPCqueue") Queue RPCqueue){
        return BindingBuilder.bind(RPCqueue).to(exchange).with("feign");
    }

    @Bean
    public Binding RpCBindingRestful(@Qualifier("directExchange") DirectExchange exchange,
                              @Qualifier("RPCqueue") Queue RPCqueue){
        return BindingBuilder.bind(RPCqueue).to(exchange).with("restful");
    }


}
```



运行消费者服务，此时RabbitMQ的管理界面可以看到：

![image-20220317143903722](springCloud.assets/image-20220317143903722.png)

同时FeignRestful 可以看到绑定routeKey

![image-20220317143925852](springCloud.assets/image-20220317143925852.png)

现在定义Listener

![image-20220317144926149](springCloud.assets/image-20220317144926149.png)

编写发布者publisher:  使用RabbitTemplate发送消息

![image-20220317145011826](springCloud.assets/image-20220317145011826.png)

成功监听到消息：

![image-20220317144859955](springCloud.assets/image-20220317144859955.png)







###### 7.6.3.2.1.7  TopicExchange 

直译： 话题类型的交换机、主题类型的交换机

![image-20220317150417527](springCloud.assets/image-20220317150417527.png)





![image-20220317150511342](springCloud.assets/image-20220317150511342.png)





例如：

![image-20220317150538969](springCloud.assets/image-20220317150538969.png)

```
如果我关心一切国家的新闻           // *.news
如果我关心中国的一切             // china.#

```





###### 7.6.3.2.1.8  TopicExchange 实战

```
topicExchange 和 directExchange的区别是：

topicExchange 支持 bindingKey 的通配符。可以为自己感兴趣的主题进行匹配。
# 表0或更多个单词
* 表1个单词
```



![image-20220317150911261](springCloud.assets/image-20220317150911261.png)



```
第一步使用Bean 来配置 交换器、队列。以及bindingKey。

创建一个叫 "crazyhh.topic"的topicExchange、 两个消费队列。

chinaAndJapanQueue :   bindingKey : {"china.#" , "japan.#"}

newsQueue  :   bindingKey :{"#.news"}
```



![image-20220317151922132](springCloud.assets/image-20220317151922132.png)

重新运行服务，查看管理页面。

找到对应Queue

![image-20220317152912218](springCloud.assets/image-20220317152912218.png)

![image-20220317152918173](springCloud.assets/image-20220317152918173.png)



![image-20220317152903350](springCloud.assets/image-20220317152903350.png)

```
china.news 被 #.news  china.# 消费
japan.news 被 #.news  china.# 消费
china.CS   被china.# 消费
japan.CS   被japan.# 消费
abc.news   被 #.news 消费
```



```
试验一下 # 匹配0个单词。
测试 “news” 和".news"

结果是都能被 "#.news"映射
```



![image-20220317154347025](springCloud.assets/image-20220317154347025.png)

















### 7.6.4  消息转换器

```
RabbitMQ 原生APi 只允许我们发送 byte[] 的数据。


Spring AMQP 可以让我们发送Object。
原因是Spring AMQP 使用了序列化器、默认使用JDk的序列化器。把Object序列化为 byte[] 但是，这是一种低效的方式。
因为jdk序列化后的文本很长，也容易发生注入。
```





![image-20220317155634448](springCloud.assets/image-20220317155634448.png)



![image-20220317155924867](springCloud.assets/image-20220317155924867.png)

```
Spring AMQP 默认使用的是JDk 序列化。这种序列化是一种低效的，容易被注入的方式。

但是Spring AMQP 支持定制系列化器。
```







![image-20220317160036447](springCloud.assets/image-20220317160036447.png)







![image-20220317160108779](springCloud.assets/image-20220317160108779.png)



引入依赖

```xml
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
```

```java
@Configuration
public class MessageConverterConfig {

    @Bean
    public MessageConverter getMsgConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```



```
publisher 和 consumer都需要配置  jackson 解析器
```



发送：

![image-20220317165855647](springCloud.assets/image-20220317165855647.png)





![image-20220317170104655](springCloud.assets/image-20220317170104655.png)



![image-20220317170047463](springCloud.assets/image-20220317170047463.png)



```
成功拿到Object
```















## 7.7 延迟队列

使用RabbitMQ延迟队列 (实现定时任务的场景)



### 7.7.1 使用场景

对于大流量的场景来说，使用柔性事务，会锁库存，将解锁订单的事件存放在消息队列中。

经过延迟队列来解锁库存。



常用解决方案：

```
spring的 schedule 定时任务轮询



RabbitMQ的消息TTL ，死信Exchange结合
```



































# 8. ElasticSerch





![image-20220317170216096](springCloud.assets/image-20220317170216096.png)





ES的各个版本文档：

https://www.elastic.co/guide/en/elasticsearch/reference/7.4/index.html



下面全部以ES:7.4为例











##  8.1 什么是ES

![image-20220317170309082](springCloud.assets/image-20220317170309082.png)

```
海量数据中搜索，分析的强项是 Elasticsearch
```







![image-20220317170723971](springCloud.assets/image-20220317170723971.png)









![image-20220317170853615](springCloud.assets/image-20220317170853615.png)





![image-20220317173354446](springCloud.assets/image-20220317173354446.png)



```
ES 对外暴露的借口是 Restful 风格，和语言无关。
```













## 8.2 ES 里的一些基本概念







![image-20220317172817834](springCloud.assets/image-20220317172817834.png)







![image-20220317173026207](springCloud.assets/image-20220317173026207.png)

![image-20220317173016647](springCloud.assets/image-20220317173016647.png)









### 8.3.1 ES 的概念和SQL概念对比



![image-20220317173051014](springCloud.assets/image-20220317173051014.png)





![image-20220518164758984](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220518164758984.png)







### 8.3.2 优势对比



![image-20220317173437615](springCloud.assets/image-20220317173437615.png)



![image-20220317173507502](springCloud.assets/image-20220317173507502.png)









### 8.3.3  DSL 是什么

ES中的查询语句称为 DSL





















## 8.3  倒排索引



对于SQL 来说，想要找到数据库中所有和手机相关的，需要这样查数据库

```
select *
from tab_store
where title like '%手机%'  
```

此时，索引失效。同时还需要比对字符串。需要一条一条比对记录，速度是O（n）非常慢。



```
ES的倒排索引，会先将 整句拆分成单词。
```



![image-20220518165518732](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220518165518732.png)

```
当检索 “红海特工行动” 时，同时命中的记录有 ： 1，2，3，4，5
那么这些记录怎么排序呢？
根据相关性得分。
```



```
记录1 : 自己有2个单词，命中了2次
记录2 : 自己有3个单词，命中了2次  //此时记录1就比记录2相关性更高
记录3： 命中2次，自己有3个单词
记录4: 命中1次
记录5 : 自己有4个单词命中了1次
```











## 8.4  docker 启动ES



```
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \ 
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
--privileged=true \
-d elasticsearch:7.4.2
```

```
如果遇到启动失败的情况，使用

docker logs <ContainerId> 查看日志
```







启动kibana



```
docker run --name kibana2 -e ELASTICSEARCH_HOSTS=http://172.17.0.3:9200  -p 5601:5601 \
--privileged=true \
-d kibana:7.4.2


```



如果出现如下错误，可以参考下面的博客

Could not create APM Agent configuration: No Living connections



https://blog.csdn.net/Jjs_Object/article/details/119873191?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-119873191-blog-105216060.pc_relevant_antiscanv2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-119873191-blog-105216060.pc_relevant_antiscanv2&utm_relevant_index=2





原因：

```
在给kibana绑定elasticsearch的ip时，这个ip并不是linux的ip，而是docker的容器对外暴露的ip

于是导致连接不上

特别说明：如果在创建运行ElasticSearch容器时没有指定ip，那么ip可能是不停变化的，以至于这次可以访问了但是下一次就不能访问，所以在创建的时候一定要指定ip: https://blog.csdn.net/Jjs_Object/article/details/119870508

解决方案
一、找到docker容器对外暴露的ip
docker inspect elasticsearch | grep IPAddress
文中的elasticsearch是容器名



执行完后可以得到一个ip  这个ip就用于给kibana绑定elasticsearch

方式一 
 在创建容器的时候绑定

docker run --name c_kibana -e ELASTICSEARCH_URL=http://172.17.0.4:9200 -p 5601:5601 -d kibana:7.4.2

 方式二
在配置文件中绑定

进入容器

docker exec -it 容器名称 /bin/bash
进入 根目录下的config目录,修改kibana,yml文件



 

在elasticsearch.hosts 这一行将ip修改成查询到的暴露ip即可


```











## 8.5 索引库

ES在存储数据时，必须要指定index索引库。索引库中有一个重要的属性  mapping  



### 8.5.1  mapping

mapping专门用于描述定义 文档。
```
例如：
定义这个index库中，String是否为全文匹配。
哪些字段包含数字，日期等格式
```

总之，类似于定义SQL的database



Mapping含有众多参数

https://www.elastic.co/guide/en/elasticsearch/reference/7.4/mapping-params.html



####  type

```
定义 字段的类型。	
```



字段的类型可以有：

```

	text: 可以被全文匹配的。(可以被分词的文本)
	keyword : 只允许精确匹配。(品牌，国家,ip地址)
	
	#除此以外，还支持数值类型
        long:
        integer
        short:
        byte:
        double:
        float
   
  	boolean
  	date
  	object
```



```
ES中没有数组。允许某个属性有多个值。ES只关心数组内元素的类型。
```









#### index

![image-20220526145615406](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220526145615406.png)



```
只有创建倒排索引以后才可以被搜索。
```









#### analyzer 

使用哪种分词器。









#### properties

```
表示某个字段的子属性
```



![image-20220526145833752](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220526145833752.png)











#### 8.5.1.1  DSL实例





![image-20220526150729318](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220526150729318.png)





### 8.5.2  查询 修改 删除数据库



删除

```
DELETE /semghh
```



![image-20220526192719905](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220526192719905.png)



```
PUT /semghh/_mapping
{
  "properties":{
    "email":{
      "type": "text"
    }
    
  }
}
```







### 8.5.3 增删改文档

增

```
POST /semghh/_doc/<docId>
{
	

}
```



删除文档

```
POST /semghh/_doc/<docId>
```


查看文档

```
GET /semghh1/_doc/<docId>
```





修改文档

```
#全量修改,删除旧文档，增加新文档
PUT /semghh/_doc/<docId>
```



```
#增量修改
POST /semghh/_update/<docId>
{
	"doc":{
		"<字段名>":<值>
	}
}
```













### 8.5.4  扁平化处理

在ES中，如果这样存储数据:

```json
{
  "group" : "fans",
  "user" : [ 
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}
```

实际上等效于这种存储

```
{
  "group" :        "fans",
  "user.first" : [ "alice", "john" ],
  "user.last" :  [ "smith", "white" ]
}
```

```
也就是说，ES默认会丢失   John-Smith 对应关系。  Alice-White 的对应关系。

只会存储  user.first中包含元素 John,alice
		 user.last中包含元素 Smith,White
		 
		 
#更通俗一点的形容,ES默认不会把数据按照对象存储。
```



那么怎么让ES记录对应关系？或者说按照对象存储呢？

```
在8.5.1中提到了 type格式 。其中有一种格式称为  nested (嵌入式)。不再让数据扁平化处理


只有在建立Index的时候声明某个字段是 nested的
```



```
建立Index的时候声明某个字段是 nested的
```

![image-20220528120753302](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528120753302.png)







#### 查询nested

在查询时比徐指明是nested查询



![image-20220528122842236](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528122842236.png)





```
在使用 
"nested":{

}
查询的时候，必须指定 “path”参数，也就是   user.first的前缀 “user"
```

























### 8.5.5  数据迁移



![image-20220528150425202](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528150425202.png)

















## 8.5  简单API

 

ES的API全部都是使用 REST 交互的。





### 8.5.1  _cat

![image-20220519105446802](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519105446802.png)















### 8.5.2  索引一个文档



![image-20220519105828741](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519105828741.png)



语法：

```
PUT   /{index}/{type}/{_id}
```



新增操作除了使用PUT 也可以使用 POST 请求：

```
POST /{index}/{type}/{_id}
POST /{index}/{type}
```



```
POST 在插入数据的时候，可以不指定 {_id} 
如果不指定{_id}则永远为新增操作。ES会为这个document赋予一个唯一的ID
如果制定了{_id},那么当{_id}对应的document不存在时，为插入操作，否则为更新操作
```



```
PUT 只允许携带{_id}，不允许省略{_id}
```













#### 8.5.2.1 示例

<img src="C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519110606813.png" alt="image-20220519110606813" style="zoom:67%;" />



```
以"_" 开头的数据，称为元数据。 使用PUT 请求插入新的数据。 当返回201表示第一次创建数据
```



<img src="C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519110753597.png" alt="image-20220519110753597" style="zoom:67%;" />



```
重复发送相同的请求变为 200 updated
同时看到  _version 版本号变为2
```





### 8.5.2 查看一个文档

```
GET  /{index}/{type}/{_id}
```











<img src="C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519111826997.png" alt="image-20220519111826997" style="zoom:50%;" />





![image-20220519112144844](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519112144844.png)

```
用于CAS的字段。如果任何其他人修改了文档，则_seq_no会+1 ,导致此时同时并发的其他人修改操作失败，重试
```





可以通过POST时指定  if_seq_no 参数，来模拟CAS并发的情况

![image-20220519112527304](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519112527304.png)





#### 8.5.2.1 示例



<img src="C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519111636831.png" alt="image-20220519111636831" style="zoom:67%;" />













### 8.5.3   更新一个文档



除了以插入时遇到相同的{_id}则变为更新。这种方式更新文档。

ES专门定义了一个接口，用于显示的声明这是一个更新操作。



语法：

```
PSOST  /{index}/{type}/{_id}/_update



{
	"doc":{
		"name":"update!"
	}

}
```





```
如果时"_update" 那么语法一定是 
"doc"{
	xxx
}
```





如果重复发一样的请求：ES不会重复更新

```
POST  _update  ，如果document没有变化则不会更新，版本号不会+1,序列号也不会+1
相应的结果是noop  “result”:"nooop"
```

![image-20220519143720645](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519143720645.png)













### 8.5.4 删除一个文档



语法：

```
DELETE   /{index}/{type}/{_id}         //删除某个文档

DELETE   /{index}                           //删除整个索引  
```







### 8.5.5 批量API    Bulk

批量操作API （并不是json）



语法:

```
POST /{index}/{typer}/_bulk
```



![image-20220519144734035](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519144734035.png)



![image-20220519144744027](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519144744027.png)









![image-20220519145119860](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519145119860.png)







<img src="C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519145141246.png" alt="image-20220519145141246" style="zoom:67%;" />



```
并不是事务,各个请求都是独立的，一个失败不会导致其他失败。
```





#### 8.5.5.1 复杂示例

![image-20220519145402501](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519145402501.png)



```
POST /_bulk 
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title": "My first blog post" }
{ "index": { "_index": "website", "_type": "blog" }}
{ "title": "My second blog post" } 
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} } 
{ "doc" : {"title" : "My updated blog post"} }


// /_bulk  没有指定任何index 指定任何type 表示是对整个ES的操作
```







```
index 是新建索引，
create 是新增文档
```









## 8.6 DSL





### 8.6.1 search





![image-20220519151615511](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519151615511.png)



![image-20220519151639288](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519151639288.png)





```
q=*                                          //表示查询所有
sort=account_number:asc                    //表示按照account_number升序排列
```





![image-20220519152109266](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519152109266.png)



![image-20220519152127731](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519152127731.png)







 ```
 _score :得分
 _source: 原本的信息
 
 虽然hit了1000条。但是默认只返回前10条记录
 ```







#### 8.6.1.1 Query DSL

ES专门定义的用于查询的语法

![image-20220519153338844](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519153338844.png)





```
 查询，使用 "query" : {}
 
 
 其中
 "query":{
 	"match_all" :{}
 }
 
 表示全匹配。
```



和Query同级的属性还有

```
"size": 20 表示hits一页的大小20个
"from": 0  表示第0页
```





#### 8.6.1.2 match

query下的属性可以有 match



语法:

```json
"match" :{
	"<fieldName>" : "<value>"
}
```

查出所有 <fieldName>中包含<value>的文档。

```
当match搜索字符串类型时,会进行全文检索。根据每条文档的相关性得分排序。
```





语法2:

```json
"match" :{
	"<fieldName>" : "<value1> <value2> ..."
}
```



假设此时 value1 = "mill"    value2= "road"

```
那么会查出  包含mill 或者 包含road 或者 包含 mill road 的全部文档
```





#### 8.6.1.3  match_phrase 

短语匹配。不再进行分词匹配。不把单词拆分，进行整个匹配。

```
"match_phrase" : {
	"<fieldName>" : "mill road"
}
```



只会匹配查询  包含 mill road的文档。不会分词匹配





#### 8.6.1.4  multi_match

多字段匹配。

语法：

```json
GET /customer/_search
{
  "query": {
    "multi_match": {
      "query": "mill",
      "fields": ["email","address"]
    }
  }
}
```

```
全文检索。 email字段中包含mill  或者  address包含mill ，满足其一就算命中文档了。
```







示例

<img src="C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220520111801466.png" alt="image-20220520111801466"  />













#### 8.6.1.5  bool 复合查询

bool 用来做复合查询： 





复合语句可以合并 任何 其它查询语句，包括复合语句。

这就意味着，复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。







```
bool 复合查询下包含几个属性

must:  必须匹配

must_not :  必须不匹配。

should:  表示应该达到的条件(可选项)。如果达到了会获得更多的相关性得分。

filter :  必须匹配。但不参与 相关性算分。
```











##### 示例1

```json
GET bank/_search 

{ 
    "query": {
        "bool" : {
            "must": [
            { "match": { "address": "mill" }},
            { "match": { "gender": "M" }} 
          ] 
        } 
 	 } 
}
```

```
查询bank索引,必须包含的条件是:  性别包含M , 地址包含mill
```







```json
GET bank/_search 
{ "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" }},
        { "match": { "gender": "M" }} 
      ],
      "should": [
        { "match": {"city": "Urie"}} ,
        {"match": {"city": "KY"}}
      ]
    } 
  } 
}
```

```
当city是Urie 或 city是ky都将会获得额外的相关性得分。
```









#### 8.6.1.6    term 精准匹配 



![image-20220526203604314](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220526203604314.png)







```json
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "age": "36"
          }
        },
        {
          "match": {
            "gender": "M"
          }
        },
        {
          "range": {
            "account_number": {
              "gte": 0,
              "lte": 1000000
            }
          }
        }
      ]
    }
  }
}


#bool 复合匹配    must必须匹配， term精准匹配， match 全文匹配  


```



```
"range": {
    "account_number": {
      "gte": 0,
      "lte": 1000000
    }


#gt 大于
#lt 小于
#gte 大于等于
#let 小于等于
```













##### terms

```
term表示精确匹配一个值。
terms 可以精确匹配一个数组中的值。
```



```json

    "terms": {
        "FIELD": [
          "VALUE1",
          "VALUE2"
        ]
    }

```



















#### 8.6.1.7  geo_bounding_box

查询 geo_point 落在某个矩形范围的所有文档



![image-20220526214102031](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220526214102031.png)







![image-20220528095632747](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528095632747.png)











#### 8.6.1.6 filter 

结果过滤

```
有一些查询并不需要计算相关性得分。仅仅是想过滤掉一些文档。可以使用filter。 免去计算相关性得分，还能调高响应效率
```



```json
GET customer/_search
{
  "query": {
    
    "bool": {
      "filter": {
        "match": {
          "address": "mill"
        }
      }
    }
  }
}
```



可以看到相关性得分为0，也就是没有计算相关性得分。

![image-20220520121810538](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220520121810538.png)



```
值得注意的是，当filter过滤掉一些文档以后，再和其他条件(must /should)进行bool复合以后，由于其他条件会计算相关性得分。最终输出的结果也会计算相关性得分。
```





官方文档这样解释filter的：

![image-20220520122035598](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220520122035598.png)

```
filter经常被用于缩小 文档上下文集合
```





### 8.6.2 相关性得分

使用match查询时，文档结果会根据词条相关度打分。







#### 8.6.2.1  functioin score query

```
使用 Function Score Query 可以修改文档的相关性得分
```





![image-20220528110524208](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528110524208.png)





```
function_score内部存在一个Query属性，可以查询任何的Query，包括bool复合查询

functions 属性即为函数，通过执行function函数，过滤出一些文档。然后为其指定 weight权重。

另外一个属性就是 boost_mode 运算模式。将相关性得分按照指定的模式于权重运算
```















```json
GET /bank/_search
{
  "query": {
    "function_score": {
      "query": {
        "bool": {
          "must": [
            {
              "term": {
                "age": "36"
              }
            },
            {
              "range": {
                "account_number": {
                  "gte": 0,
                  "lte": 1000000
                }
              }
            }
          ]
        }
      },
      "functions": [
        {
          "filter": {
            "range": {
              "account_number": {
                "gte": 10,
                "lte": 20
              }
            }
          },
          "weight": 2
        }
      ],
      "boost_mode": "multiply"
    }
  }
}
```









### 8.6.3  sort 排序

```
使用sort 对ES查询出的结果，进行排序。
```





![image-20220528112928769](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528112928769.png)





```
sort 可以传入多个排序标准。
```





![image-20220528113237029](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528113237029.png)



```
先按评分降序，再按价格升序。  #通常会带上评分降序，因为评分高是相关度高的直接体现。
```







### 8.6.4 highlight 高亮处理

```
高亮就是着重显示。


在h5页面中，可以通过行内样式来改变某一个元素的样式。通常ES的返回值会直接被前端显示。
原本返回数据 "ab" ,通过高亮处理后  "<h1>ab</h1>" 。前端页面将会直接将以h1级标题显示。

这种处理称为  高亮。 ES支持对指定字段，加上指定的高亮处理。
```



![image-20220528113948346](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528113948346.png)



```
pre_tags : 前缀

post_tags: 后缀
```







返回的结果：





















### 8.6.5 Aggregation

```
聚合(Aggregation)  可以将搜索出的数据，总结为某种指标。 甚至是统计数据。用于数据分析
例如求平均数，
```



官方参考文档：

https://www.elastic.co/guide/en/elasticsearch/reference/8.2/search-aggregations-metrics.html



聚合分为3类

```
指标聚合(Metrics aggregations)  : 指标聚合指的就是计算各种统计值。 例如总和，平均值等

桶聚合 (Bucket aggregations):   桶聚合就是 按照文档的字段值，范围将文档分到不同组中。

管道聚合 (Pipeline aggregations)  :  从其他聚合中获取数据，而不是从文档中获取
```





#### 8.6.2.1  统计聚合



##### avg



语法

```js
{
	...
	"aggs": {
        "<aggName>" :{
            "avg":{ "field" : "<fieldName>" }
        }
    }
}
```



```
在上述语法中 ，aggName 是给这个聚合起一个名字。  fieldName 指的是需要被统计的字段名


avg 值得是聚合的种类，根据不同的聚合，有不同的值。
```



##### Max



```json
GET /bank/_search

{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "size": 20,
  "from": 0,
  "aggs": {
    "max_price": {
      "max": {
        "field": "age"
      }
    }
  }
}
```











##### Min

```json
GET /bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "size": 2,
  "from": 0,
  "aggs": {
    "min_price1": {
      "min": {
        "field": "age"
      }
    }
  }
}
```



他的响应是这样的

![image-20220519212534641](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519212534641.png)

```
花费
是否超时
分片信息
命中集
聚合
```





##### Percentiles 



百分比分布 聚合。    解释：

```
默认有 1 5 25 50 75 95 99 个维度。分别表示超越了1%的值为5，超越了5%的数值为25  。或者说值为985超越了99%

            "1.0": 5.0,
            "5.0": 25.0,
            "25.0": 165.0,
            "50.0": 445.0,
            "75.0": 725.0,
            "95.0": 945.0,
            "99.0": 985.0
```



请求语法：

```json
GET latency/_search
{
    "size": 0,
    "aggs" : {
        "<aggName>" : {
            "percentiles" : {
                "field" : <fieldName>,
                "percents" : [10,50,90] 
            }
        }
    }
}
```

```
表示获取指定 <fieldName>字段的 [10,50,90]分布
```



![image-20220520084635147](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220520084635147.png)









##### percentile ranks

反过来，求固定值所占的百分比



请求格式

```json
GET latency/_search
{
    "size": 0,
    "aggs" : {
        "load_time_ranks" : {
            "percentile_ranks" : {
                "field" : "<fieldName>", 
                "values" : [500, 600]
            }
        }
    }
}
```







![image-20220520084901614](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220520084901614.png)





##### sum 

​	

```json
GET /bank/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "match" : {
          "address" : "mill"
        }
      }
    }
  },
  "aggs": {
    "age_sum": {
      "sum": {
        "field": "age"
      }
    }
  }
}
```



#### 8.6.2.2 Bucket聚合



##### terms 

根据指定字段的值作为分类，相同值的分为一组。

并且分别计算了每个桶内落入的文档数目。





















参考博客

https://blog.csdn.net/qq_18218071/article/details/118858274





















#### 8.6.2.3  子聚合



通过聚合可以按照某种规则，将查出的结果分类。

如下搜索： 表示按照brandId对结果进行分类。

```json
GET /gulimall_product/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "brand_id_agg": {
      "terms": {
        "field": "brandId",
        "size": 10
      }
    }
  }
}
```



查到的结果如下：

![image-20220528164129396](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528164129396.png)

```
表示BrandId 一共有2种：  key=12 命中了18个文档  key=9命中了8个文档
```

那么如果此时想知道key=12的这条文档的其他字段信息怎么办？

```
使用子聚合.

子聚合可以使用父类聚合分类的bucket
```



子聚合查询如下：

```json
GET /gulimall_product/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "brand_id_agg": {
      "terms": {
        "field": "brandId",
        "size": 10
      },
      "aggs": {
        "brand_name_agg": {
          "terms": {
            "field": "brandName",
            "size": 10
          }
        }
      }
    }
  }
}
```





如果同时想知道brandName 和 brandImg字段怎么办？聚合支持多个并在一起

![image-20220528165647843](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528165647843.png)

```
平级的两个聚合。

也可以再使用子聚合嵌套。
```



部分代码：

```
...
	"aggs": {
        "brand_name_agg": {
          "terms": {
            "field": "brandName",
            "size": 10
          }
        },
        "brand_image_agg":{
          "terms": {
            "field": "brandImg",
            "size": 10
          }
        }
      }
...
```



































## 8.7 RestHighLevelClient



Java可以使用 RestHighLevelClient来连接ES客户端。并进行DSL查询。



`rest high level client` 可以说是使用java来连接ES的一套标准。 官方参考文档如下：

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.4/java-rest-high-document-index.html



java文档手册

https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-high-level-client/7.8.0/index.html













使用前首先引入依赖


```xml
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.4.2</version>
        </dependency>
```







可以向Spring容器中，注入一个全局唯一的Client . Bean的配置如下

```java
@Configuration
public class GulimallElasticConfig {

    public static final RequestOptions COMMON_OPTION;

    static {
        RequestOptions.Builder builder = RequestOptions.DEFAULT.toBuilder();
        COMMON_OPTION =builder.build();
    }
    @Bean
    public RestHighLevelClient esRestClient(){
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("192.168.202.137",9200,"http")
                ));
        return client;
    }
}
```



```
只得注意的是，上述配置是单机模式。并非集群模式，集群模式的配置详见官方文档。
```















在启动SpringBoot时如果报数据源错误，整个module又不用数据库操作。就可以在自动配置时，去掉数据源的自动配置。

```
DataSourceAutoConfiguration.class
```



![image-20220519164947248](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519164947248.png)











### 8.7.1 Index API







#### 8.7.1.1 RequestOptions

```
请求的选项。 可以对所有的Request做统一设置。

官方建议可以给RequestOptions建立一个单实例的
```





![image-20220519165944053](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519165944053.png)





#### 8.7.1.2  创建index

参考文档

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.4/java-rest-high-create-index.html



<img src="C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519170925497.png" alt="image-20220519170925497" style="zoom: 80%;" />



```
创建一个带有Index名称的 IndexRequest

可以有多种方式填写Index请求的内容。 在官方的文档中有详细介绍。
```



```java
        IndexRequest request= new IndexRequest("users");
        request.id("1");
//        request.source("userName","张三")
//                .source("age",18)
//                .source("gender","男");

        User user = new User("张三",18,"");
        String jsonString = JSON.toJSONString(user);
        request.source(jsonString, XContentType.JSON); //必须指定内容类型 JSON


//创建了一个IndexRequest ，指定了index，并且传入了source
```

Rest High Level Client提供了 两种提交到ES Server的办法：同步提交，异步提交

```
```



![image-20220519172023871](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519172023871.png)





异步的方式

![image-20220519174532546](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519174532546.png)















##### 8.7.1.2.1 可选参数



![image-20220519175502945](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519175502945.png)

```
设置超时时间。
```







![image-20220519175916446](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519175916446.png)

```
Index Response 可以拿到一些回调信息。比ES server 是否接收信息。
```





#### 8.7.1.3  删除Index



```java
@Test
public void testForDeleteIndex() throws IOException {
    DeleteIndexRequest req = new DeleteIndexRequest("users");
    AcknowledgedResponse resp = client.indices().delete(req, COMMON_OPTION);
    System.out.println(resp);

}
```



同步执行：

![image-20220519181121141](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519181121141.png)

异步执行：

![image-20220519181131528](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519181131528.png)





#### 8.7.1.4 Index 是否存在



![image-20220519182009302](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519182009302.png)









```java
@Test
public void testForGetIndex() throws IOException {

    GetIndexRequest req = new GetIndexRequest("customer","users");
    boolean exists = client.indices().exists(req, COMMON_OPTION);
    System.out.println(exists);
}
```



```
可以同时查询多个 indices ，只要有1个不存在，就返回false 
全都存在才返回true
```







#### 8.7.1.5 打开/关闭 index



什么是打开/关闭索引？

```
索引被关闭，那么这个索引只能显示元数据信息，不能够进行读写操作。
```









![image-20220519183029054](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519183029054.png)

![image-20220519183021611](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519183021611.png)







![image-20220519183039973](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519183039973.png)





![image-20220519183054540](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519183054540.png)



#### 8.7.1.6  克隆index



![image-20220519201313884](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519201313884.png)



```
同步方法 clone
```



![image-20220519201359260](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220519201359260.png)









### 8.7.2  search API

```
查询相关的API
```



#### 8.7.2.1 RestHighLevelClient.search()



![image-20220528232033485](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528232033485.png)

```
常用的查询方法。Client.search() 传入一个SearchRequest 和 RequestOption 

其中Option使用全局唯一的Bean
```



##### SearchRequest

直接new 一个SearchRequest对象

```java
SearchRequest searchRequest = new SearchRequest("bank"); //传入 index库的名
```



那么Search的具体Query语句在哪里写?

SearchRequest.source() 方法传入一个SearchSourceBuilder类对象。而整个SearchSourceBuilder就是用于构建DSL的

```java
searchRequest.source(searchSourceBuilder); //传入一个sourceBuilder
```





#### 8.7.2.2  SearchSourceBuilder



RestHighLevelClietn 这套API ,最常用的两个思想就是  工具类 ，和 Builder 。在构建整个DSL的过程中，使用了大量以s结尾的工具类(类似于JDK中的Executors) 和 Builder构造类。





SearchSourceBuilder就是DSL的根。

```java
searchSourceBuilder.query(QueryBuilder);//传入一个Query的构造器
searchSourceBuilder.aggregation(AggregationBuilder);  //传入一个聚合的构造器
searchSourceBuilder.sort(SortBuilder);  //传入一个sort排序的构造器
searchSourceBuilder.from(int);//传入分页的起始索引
searchSourceBuilder.size(int);//传入分页的长度
searchSourceBuilder.highlighter(highlighteBuilder); //传入一个高光构造器
```



可以看到，几乎所有复杂的属性，都配有对应的Builder. 这些Builder绝大部分都有对应的工具类，例如：

```java
QueryBuilders
AggregationBuilders
SortBuilders

//这些工具类全部支持链式调用。
```







##### toString()

SearchSourceBuilder用于构建查询语句，最终的查询条件都会绑定在SearchSourceBuilder中。

当构建完查询语句后，可以使用SearchSourceBuilder.toString()语句查看最终的查询json



















#### 8.7.2.3 QueryBuilders



QueryBuilders最常用的工具类。用于构造各种Query查询。

```
包含的方法几乎全都是见名知意; 按照客户端的DSL语句几乎可以完全复刻出来。
例如 
boolQuery()
nestedQuery()
termQuery()
matchQuery()
matchAllQuery()
```

![image-20220528234035797](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528234035797.png)









##### boolQuery



复合嵌套查询，在复杂的query语句中必不可少的条件。

```
方法名很直观
```







![image-20220528233750941](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528233750941.png)













#### 8.7.2.4 AggregationBuilders

和QueryBuilders一样，都是工具类，用于构造聚合函数的。





```

```



![image-20220528234417824](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528234417824.png)













#### 8.7.2.5  SortBuilders





![image-20220528234551748](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528234551748.png)









##### 示例

```java
SortBuilders.fieldSort("brandId").order(SortOrder.DESC); //根据brandId字段降序排序
```















### 8.7.3  Response API

这一小节，用于解析RestHighLevelClient返回后的数据



参考官方文档

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.4/java-rest-high-search.html#java-rest-high-search-response











#### 8.7.3.1   SearchResponse

client.search()返回的对象类型 ：  SearchResponse

```
DSL语句返回的根对象。
```





常用的几个API

```
getHits()//最常用的，用于获得命中集
getAggregations()  //获得聚合
getTook() //获得花费时间
```



![image-20220528234839496](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528234839496.png)









#### 8.7.3.2  SearchHits

命中集。对应的是 "hits"部分结构

![image-20220530102108532](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220530102108532.png)







SearchHits包含了全部的 SearchHit (一个SearchHit 就相当于命中了一个文档。)

![image-20220528235032898](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528235032898.png)











#### 8.7.3.3  SearchHit

```
每一个命中的文档元信息，存放在SearchHit中。
```

元信息有哪些？

![image-20220530112916482](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220530112916482.png)

```
_index
_type
_id
_score
_source   //这个文档的全部源信息
```









方法如下



![image-20220528235319629](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220528235319629.png)



常用的API

```
getSourceAsMap()  //以键值对的形式拿到全部的_source
```

















#### 8.7.3.4  Aggregations 

这个类并不是聚合的工具类，而是表示 多个聚合的集合。

![image-20220530120752956](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220530120752956.png)

```
这个类的目的就是获得整个聚合。 提供了一些方法，来获得包含的每一个聚合。或者遍历聚合
```





##### 成员变量

![image-20220530121018877](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220530121018877.png)

```
显然DSL语句返回的根聚合都会放入其中。
```







##### 常用的方法



![image-20220530121204441](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220530121204441.png)









```
get(); //根据聚合名称，找到指定聚合
```

![image-20220530121240323](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220530121240323.png)







```
getAsMap(); 以Map的方式拿到全部的聚合
```



![image-20220530121259097](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220530121259097.png)











#### 8.7.3.5  Aggregation

Aggregation是一个非常复杂的接口。有非常多的实现类。



https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.4/java-rest-high-search.html#java-rest-high-search-response-aggs





一个接收返回聚合的Java代码如下：

![image-20220530124005210](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220530124005210.png)



```
在1处，获得名为"by_company"的聚合,必须为其指明返回的数据类型是  Terms
```



总的来讲，必须熟悉DSL语句中Aggregation的种类。在Java中都有其对应的Java类。

```
例如Terms
```











##### 8.7.3.5.1  Terms

```
Terms 接口用于表示  DSL中  terms聚合。
```



源码解释：

```
Terms接口定义了一个多值的桶结构。每一个桶都代表指定字段(terms聚合必须指定一个字段)独一无二的取值。
所有的文档都会落入对应的桶内。
```



![image-20220530124721986](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220530124721986.png)



常用的方法：

```
getBuckets() 获得全部的桶
```

























## 8.8 IK分词器	

分词的逻辑是在分词器内有一个字典。对字典里的词语进行匹配。



ik分词器的github库

https://github.com/medcl/elasticsearch-analysis-ik



### 8.8.1 词库扩展





![image-20220526142907295](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220526142907295.png)



#### 扩展 本地可用字典

#### 扩展 本地停用字典

#### 支持 远程字典







### 8.8.2. 热更新词库



![image-20220526143022189](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220526143022189.png)























# 9. 阿里云OSS

OSS对象存储。

由于集群服务，如果是服务本地存储，那么下一次请求的服务器不是同一个，则会出现找不到文件的情况。

![image-20220514220120673](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220514220120673.png)

```
下一次如果访问的是其他服务器，则找不到文件。
```



对象存储: 

![image-20220514220149182](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220514220149182.png)

```
上传的文件不存放在服务本地。而是存放在专门的文件存储中。由文件存储返回URL
```







## 9.1 阿里云SDK

帮助文档 https://help.aliyun.com/document_detail/32009.html



导入SDK依赖

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.10.2</version>
</dependency>
```



如果使用的是Java 9及以上的版本，则需要添加jaxb相关依赖。添加jaxb相关依赖示例代码如下：

```xml
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.1</version>
</dependency>
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
</dependency>
<!-- no more than 2.3.3-->
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
    <version>2.3.3</version>
</dependency>
```





## 9.2 SpringCloud alibaba 

SpringCloud alibaba oss 可以更快速使用开发OSS



引入依赖

```xml
<dependency>
	<groupId></groupId>
	<artifactId>spring-cloud-starter-alicloud-oss</artifactId>
</dependency>
```







![image-20220515095449354](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515095449354.png)













# 10. JSR303 校验

 给Entity加上校验注解 即完成校验



首先需要引入依赖

```xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
```



@NotBlank 

```
不能为null 也不能是空串。 至少有1个字符	
```


@Valid

```
开启校验注解。
```



更多的校验注解在这个包下



![image-20220515123531787](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515123531787.png)



## 10.1 message属性

在校验注解下存在一个属性 ： String message()。用于当校验不通过时，返回的信息。

![image-20220515123731162](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515123731162.png)

```
动态返回指定message下的语句。 这些语句具体是什么，配置在了ValidationMessage.properties中
```

![image-20220515123843121](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515123843121.png)



除此以外，我们还可以自定义message信息。例如：

![image-20220515123953584](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515123953584.png)



此时响应结果如下：

![image-20220515124731351](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515124731351.png)

```
可以看到defaultMessage 变成了我们自定义的message。
但，这不符合我们的业务逻辑。
我们需要 {code,msg,data}这样的返回形式。后面我们继续处理
```





## 10.2 BindingResult

在紧跟着被校验的形参，传入一个BindingResult 可以获取到校验结果

```
BindingResult 可以拿到全部的校验异常。作为List返回
```

![image-20220515125923046](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515125923046.png)








```
FieldError  中可以拿到 具体是哪个字段过不了校验。 过不了的值(RejectedValue) 具体是多少
```



![image-20220515125840080](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515125840080.png)



此时我们就可以自定义我们想要的结果了：

```java
    @RequestMapping("/save")
    public R save(@Valid @RequestBody BrandEntity brand,BindingResult result){

        if (result.hasErrors()) {
            R r = R.error(400,"校验结果出错");
            List<FieldError> fieldErrors = result.getFieldErrors();
            HashMap<String,Object> data = new HashMap<>();

            fieldErrors.forEach(x->{
                data.put(x.getField(),x.getRejectedValue());
            });
            r.put("data",data);
            return r;
        }else {
            brandService.save(brand);
        }

        return R.ok();
    }
```

![image-20220515131103939](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515131103939.png)









## 10.3 更多的注解



### @URL





### @Pattern

```
可以传入一个正则表达式，满足正则表达式才能通过校验。
```

![image-20220515132647147](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515132647147.png)





## 10.4 统一异常处理

使用SpringMVC 的 `@ControllerAdvice`注解

![image-20220515133214482](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515133214482.png)

当不使用BindingResult 捕获了这些异常。那么这些异常就会被抛出去。

被 `@ControllerAdvice`接收并处理.

由专门的异常处理类 提供策略来管理这些异常：

```
要么是将异常抛出给用户，
要么是重试等
```



### 10.4.1 定义异常处理类

首先，集中处理异常的类，需要标注 `@ControllerAdvice`

![image-20220515135949733](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515135949733.png)

```
如果需要日志打印的话，可以标注@Slf4j 并引入对应的日志框架



@ControllerAdvice注解的属性-basePackages: 表示处理指定包下的异常。
```



### 10.4.2 给处理方法标注@ExceptionHandler

在处理异常的方法中，使用 `@ExceptionHandler`指明要处理的异常种类。

![image-20220515140023991](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515140023991.png)



`MethodArgumentNotValidException`方法参数不合法异常，是Spring提供的一种异常。

当controller方法参数校验不成功时，会抛出这个异常。由于我的代码引入了校验规则，所以当校验出现异常时会抛出对应的异常。

所以统一异常处理时需要对应的处理方法。

```
异常处理的返回值将作为，Controller的返回值。所以是 R
我们需要把R JSON化，所以使用@ResponseBody注解
```



```
具体方法内的逻辑，由程序员编写。满足统一业务需求即可
```











```java
@Slf4j
@ControllerAdvice(basePackages = "com.atguigu.gulimall.product.controller")
public class GulimallExceptionControllerAdvice {


    @ResponseBody
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public R handlerValidException(MethodArgumentNotValidException e){

        log.error("数据校验出现异常{},异常类型{}",e.getMessage(),e.getClass());
        BindingResult bindingResult = e.getBindingResult();

        HashMap<String,String> data = new HashMap<>();
        for (FieldError fieldError : bindingResult.getFieldErrors()) {
            data.put(fieldError.getField(), fieldError.getDefaultMessage());
        }

        return R.error(400,"参数校验失败").put("data",data);
    }

}
```





使用了集中异常处理以后，Controller层就变得非常清爽：

![image-20220515153316589](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515153316589.png)







### 10.4.1  统一错误码

![image-20220515140428091](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515140428091.png)





![image-20220515140547347](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515140547347.png)



```
当业务量大了以后，各个业务模块需要使用统一的 错误码。

此时可以把错误码定义在 common模块中，因为众多模块都需要引用它
```



```java
public enum BizCodeEnume {
    UNKNOW_EXCEPTION(10000,"系统未知异常"),
    VAILD_EXCEPTION(10001,"参数格式校验失败");

    private int code;
    private String msg;
    BizCodeEnume(int code,String msg){
        this.code = code;
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}
```











## 10.5 分组校验

有这样一个场景：对于自增主键 id

```
当插入一个新的数据时，我们不希望带id参数。
当修改一条记录时，我们希望必须带id参数
```

此时怎么办？使用分组校验。



另外的场景：

```
修改某条记录时，我们希望id总是不为null, 为null的其他字段总是不需要修改的字段。不为null的字段总是需要修改的字段
```

使用：分组校验



怎么使用？

```
注解包含一个属性  groups. 指定groups 即可指定不同的分组。
```



![image-20220515144828989](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515144828989.png)

```
groups 必须是一个类型，而且必须是一个接口
```



Add操作，Update操作，属于各种微服务都可以使用的分组。所以，我们可以定义在common模块中

![image-20220515145025842](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515145025842.png)



怎么告知Controller使用哪个分组校验呢？

```
使用@Validated 注解
```

![image-20220515153246613](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515153246613.png)



### 10.5.1  不指定分组的校验不起作用

```
不指定分组的校验不起作用.所以,校验都必须指定分组。
```







## 10.6 自定义校验



```
1.自定义一个  注解
2.自定义一个  校验器
3. 将校验器和注解关联
```



在Valid包下。创建一个ListValue注解。用于判断字段的值，是否在指定的value数组中

![image-20220515160100814](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515160100814.png)





JSR303规定Valid注解必须包含以下3个属性

```java
    String message() default "{javax.validation.constraints.Null.message}";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

//出错的信息
//分组
//额外的挂载信息
```



![image-20220515160147517](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515160147517.png)



```
其中 Message的 default信息默认作为key在ValidationMessage.properties 中搜索value值。
所以可以在资源路径下创建一个 ValidationMessage.properties 中配置返回信息。
```

![image-20220515161432105](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515161432105.png)







自定义校验注解需要以下的元注解

```java
@Target({ElementType.FIELD})   //表示能哪些位置能被标注
@Retention(RetentionPolicy.RUNTIME) //运行时注解
@Documented    
@Constraint(validatedBy = {ListValueConstraintValidator.class})   //用于关联自定义校验器
```















### 10.6.1 自定义校验器



光有注解没有业务逻辑，注解是不会生效的。校验注解 和 校验器需要一一对应。



自定义校验器需要实现以下接口

```java
package javax.validation;

import java.lang.annotation.Annotation;

public interface ConstraintValidator<A extends Annotation, T> {
    
    default void initialize(A constraintAnnotation) {
    }

    boolean isValid(T var1, ConstraintValidatorContext var2);
    
}
```



```
其中，A 是需要校验的注解。T是需要校验数据的类型。
```



ListValue注解校验器

```java
public class ListValueConstraintValidator implements ConstraintValidator<ListValue,Integer> {

    HashSet<Integer> set =new HashSet<>();
    @Override
    public void initialize(ListValue constraintAnnotation) {


        int[] vals = constraintAnnotation.vals();
        for (int val : vals) {
            set.add(val);
        }
    }

    @Override
    public boolean isValid(Integer integer, ConstraintValidatorContext constraintValidatorContext) {
        if (integer==null)return true;
        return set.contains(integer);
    }
}
```



最终在注解中关联起来

![image-20220515162238375](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515162238375.png)







```java
//自定义校验器是可以被IOC容器注入对象的
public class CropNameValidConstraint implements ConstraintValidator<CropNameValid,String> {


    @Autowired
    private CropTypesService service;


    @Override
    public boolean isValid(String s, ConstraintValidatorContext constraintValidatorContext) {
        return service.existCropType(s);
    }
}
```













### 10.6.2 更多的校验器



同一个注解，可以根据校验器适配的类型不同，而配置多个校验器例如下图：

![image-20220515162346447](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515162346447.png)

```
这个注解，有Byte校验器，Number校验器等等
```







# 11. SPU SKU



![image-20220515194812463](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515194812463.png)

用于快速检索的单位，检索到以后，将获取更进一步的 `sku`



```
spu : 苹果11
```













![image-20220515195755104](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220515195755104.png)

```
sku: 苹果11  16G 土豪金
```

















# 12. Object划分



##   PO 持久对象

persistant Object 

![image-20220517202743381](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220517202743381.png)



也就是Entity



##  DO  领域对象

Domain Object

![image-20220517202826508](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220517202826508.png)

抽取业务的概念，形成的对象，都可以称之为DO





## TO  数据传输对象

transfer object  数据传输对象

 ![image-20220517202945271](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220517202945271.png)



```
例如微服务之间相互调用，被封装和发送的传输对象。就可以称之为 TO
```





## DTO 数据传输对象

和TO一样



## VO   值对象

value Object 

![image-20220517203148815](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220517203148815.png)



```
根据具体业务，创造出的对象。这些对象通常不是Entity 因为他们可能只是Entity的一部分，并不完整，或者超越了某一个Entity，包括了其他Entity的部分字段。


或者可以理解为  View Object
```





作用：

```
接收页面传递来的数据，封装对象。将业务处理完成的对象，封装成页面要用的数据，传输出去。
```







## BO  业务对象



![image-20220517203818857](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220517203818857.png)











BeanUtils.copyProperties(Obejct source,Obejct  target);

可以将源Bean的属性，复制到 target中 。常用于 VO，PO之间属性的转换







# 13. 压测



## 13.1 压测指标



### Response time 

响应时间



### TPS 

Transaction per Second  每秒处理的事务数量。

```
不单单指数据库上狭义的事务。 一般指一个完整的整个业务流程
```



### QPS 

query per Second  每秒查询的数量









### 最大响应时间



### 最小响应时间



### 90%响应时间







![image-20220521171442685](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220521171442685.png)







## 13.2 衡量标准

大致如下:



![image-20220521170912003](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220521170912003.png)





## 13.3  结论

中间件越多，吞吐量损失越多。大多数都损失在中间件之间的交互了。



可能称为性能瓶颈的点：



DB  (加索引,加缓存)

模板引擎渲染 (开缓存)

静态资源 (动静分离)  使用nginx做动静分离





# 14.  缓存与分布式锁



使用缓存

 ![image-20220521211659964](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220521211659964.png)





## 14.1 哪些数据适合写入缓存



1. 即时性，数据一致性要求不高的数据
2. 访问量大，反复多次查询，会产生嵌套查询的。更新频率不高的





## 14.2 本地缓存



![image-20220521212732157](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220521212732157.png)

```
本地缓存指的是 缓存保存在JVM里。

这样的缓存速度最快。但容易消耗JVM内存。 如果是分布式集群，浪费更多的缓存，同时会造成一定程度的数据不一致。
```



所以应该使用公共的缓存中间件。







## 14.3 加锁解决缓存击穿



```
加分布式锁，保证更新缓存时只有1个线程能查数据库。其他的线程通过等待重试，查缓存。
```





分布式锁详情参考Redis的笔记







## 14.4  SpringCache



SpringCache希望使用统一的接口来管理缓存。使用注解就省去了繁琐的格式化取存缓存操作。

注解前瞻：

![image-20220523114225536](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220523114225536.png)





### 14.4.1 基本概念











# 15. 分布式Session





## 15.1  传统Session分布式下的问题



在传统Session中会遇到2个方面的问题：

```
Session在各自服务器的内存下，集群中无法共享。

其他微服务无法共享
```









## 15.2 解决方案

```
使用Spring Session
```







![image-20220606150013945](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220606150013945.png)







## 15.3  使用 Spring Session 



参考官方文档

https://docs.spring.io/spring-session/reference/guides/boot-redis.html

### 15.3.1  引入依赖

```xml
<dependencies>
	<!-- ... -->

	<dependency>
		<groupId>org.springframework.session</groupId>
		<artifactId>spring-session-data-redis</artifactId>
	</dependency>
</dependencies>
```





### 15.3.2 配置Session保存

```
配置Session保存在Redis里
```



![image-20220606212959338](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220606212959338.png)







更多的配置：

![image-20220606212942559](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220606212942559.png)

```
Session 的过期时间
Session的刷新策略
Session的名称空间
```







参考

 [Spring Session](https://docs.spring.io/spring-boot/docs/2.7.0-M3/reference/htmlsingle/#boot-features-session) 





### 15.3.3 更多配置

#### 配置domain 

配置Cookie的Domain,需要自定义Cookie 。根据官方文档，注入一个CookieSerializer

```java
    @Bean
    public CookieSerializer getCookieSerializer(){
        DefaultCookieSerializer cookieSerializer = new DefaultCookieSerializer();

        cookieSerializer.setDomainName(".gulimal.com");
        return cookieSerializer;
    }
```









#### 配置JSON序列化

使用JSON作为Redis中存储Session的序列化方式。同样的，注入一个RedisSerializer

```java
@Bean
public RedisSerializer<Object> getRedisSerializer(){
    return new GenericJackson2JsonRedisSerializer();
}
```







### 15.3.4 spring session原理



![image-20220606224903697](C:\Users\SemgHH\Desktop\笔记\springCloud.assets\image-20220606224903697.png)





```
1.首先注入了一个 SessionRepository  专门用于获取Session的DAO
2.如果是Redis作为存储Session介质，则会使用相关的  Repository ,用于Session的增删改查.
3.添加了一个过滤器(SessionRepositoryFilter) 继承了java.Servlet.Filter原生的Filter，对所有的请求进行过滤。
4.把所有的HttpRequest 和HttpResponse进行包装(实现了原生的)
```









# 16. 分布式事务





## 16.1 CAP

Raft理论











## 16.2  BASE理论

对CAP的延申，如果无法做到强一致，可以牺牲强一致，使用最终一致性





### 16.2.1 基本可用

basically available



在分布式系统出现故障时，允许损失部分可用性 (响应时间, 部分非核心功能的可用性)。 但是不等价于整个系统不可用。



```
响应上的损失:   正常情况需要在0.5秒内返回给用户结果，由于出现故障，查询响应时间变为1~2秒


非核心功能的损失:   在流量高峰时，为了保护系统稳定，部分消费者可能会使用降级服务
```





​	

### 16.2.2 软状态

```
允许系统中出现中间状态。这个中间状态不会影响系统整体的可用性。
```





### 16.2.3 最终一致性

```
系统中所有数据副本经过一定时间后，最终能达到一致的状态。
```















## 16.3 分布式事务的几种方案



### 16.3.1  2pc

```
两阶段提交，也叫做  XA Transactions
```



```
第一阶段预提交 。          有任何一个数据库拒绝提交，所有的数据库都会回滚此次事务。

第二阶段真正提交(保存更改)
```







#### 16.3.1.1 XA的特点



```
XA 协议简单，数据库原生支持。 使用分布式事务的成本也比较低

XA 性能不理想， 在大流量高并发场景下性能不好




```















### 16.3.2   3pc

//TODO









## 16.4    柔性事务 TCC

TCC事务补偿型方案



```
刚性事务： 遵循ACID 

柔性事务： 遵循BASE理论


柔性事务允许一定时间内，不同节点的数据不一致，但要求最终一致性。
```





TCC要求实现一个功能需要实现3个接口

```
try               : 预准备数据

confirm           : 提交

cancel            : 回滚提交的数据
```





![image-20220810112414783](springCloud.assets/image-20220810112414783.png)









## 16.5 柔性事务-最大努力通知



```
按规律进行通知,不保证数据1次一定成功，但会提供可查询操作接口进行核对。在进行某种操作时，按频率通知，直到对方回复解决成功。

这种方案通常用在与第三方系统通讯时。比如调用 微信，支付宝支付后的支付结果通知。 通常结合MQ进行实现。


银行通知，商户通知(各大交易业务平台间的商户通知,多次通知，查询校对，对账文件等)
```

























































































# 17. 微服务划分原则

参考 https://www.jianshu.com/p/1c16195e9a1b



## 17.1 模块划分原则



1.单一服务内功能高内聚，低耦合



2.微服务闭包原则 ：

```
当我们改变一个微服务的时，所有依赖都在这个微服务组件内，不需要修改其他微服务。
```



3.服务自治，接口隔离：

```
尽量消除对其他服务的强依赖，这样可以降低沟通成本，提升服务稳定性。



服务通过标准的接口隔离，隐藏内部实现细节。这样可以使得服务 独立开发，测试，部署，运行。
```



4.优先剥离边缘业务

通常微服务的拆分，可能在开发过程中。 也就是说产品一边迭代，一边完成微服务的拆分。

通常会优先剥离边缘业务：例如短信服务等。



5.避免环形依赖





## 17.2 拆分策略

按照`功能` `非功能` 维度进行考虑。

功能维度：

```
主要划分清楚业务的边界
```

例如：

 DDD 领域驱动设计





非功能维度:

```
DDD 领域驱动设计
```













