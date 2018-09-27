https://github.com/chxfantasy/spring-cloud-demo.git

前奏https://www.jianshu.com/p/c14c47243994
创业半年多，中间做过一个app。各种各样的原因，最后虽然上线了，但是没有推广，每天百十来个用户，也不打算迭代了。我们后台使用的是基于spring-boot、spring-cloud的微服务框架。框架包含了spring-cloud全家桶的大部分组件，我自己花时间，重新把这些框架类的内容整理了一遍，构建了一个简单的场景，开源出来，希望对那些想学习 或者 快速搭建自己的微服务框架的同学能提供帮助。
代码地址(附上原文地址)：https://github.com/chxfantasy/spring-cloud-demo
截图：

代码目录
如果对微服务本身不太太了解的话，或者不知道为什么需要微服务的话，请参考原来的一篇文章：我也想写篇微服务的文章，以及微服务的优缺点
如果对微服务里面的各个组件不太熟悉的话，可以参考这一篇文章：spring cloud从看不懂到放弃
项目介绍
1.这仅仅是spring-cloud微服务框架的一个demo，可以完整运行。可能你初看到这里，会觉得这个项目目录太tm复杂了，确实，微服务要做的就是解耦、轻量化。这里面包含的组件和内容有: 
1.spring cloud eureka，服务注册和服务发现
2.spring cloud config，动态配置项
3.ribbon，客户端负载均衡
4.feign，
5.hystrix，熔断
6.turbine
7.Spring Cloud Starters
8.同一个服务中的多数据库支持（AOP）
9.全链路traceId追踪
10.velocity 前端模板
11.mybatis, pageHelper (分页), druid (连接池)
12.redis（序列化采用的是jdk默认序列化方案）
13.slf4j & logback(及其配置)
14.国际化配置
15.全局错误信息catch
16.线程池
17.服务健康检查, 服务全链路健康检查
18.等
2.代码的业务是：首先有一个登录页面，用户登录以后，能看到一个微博(moment)列表，同时也可以添加微博。在每个微博的最右边，可以看到每个微博的评论列表，也能添加评论。前端代码写的非常粗糙。。。 如果你真的运行了，求不吐槽。
3.这里不谈根据业务的需求去选择是否微服务化，也不争辩什么时候该用什么样的框架，我这里只是把我们app后端的代码，抽象成一个代码库，希望能为你提供帮助。我们产品上线2个多月，后端基本没有出过问题(也跟量小有关系)。
运行
1.首先，你在本地需要有一个redis和mysql，redis默认启动就可以。数据库表的创建语句见下：
create database test;

CREATE TABLE `account` (
  `user_id` varchar(127) NOT NULL DEFAULT '',
  `user_name` varchar(127) NOT NULL DEFAULT '',
  `password` varchar(127) NOT NULL DEFAULT '',
  `gmt_created` datetime NOT NULL,
  `gmt_modified` datetime DEFAULT NULL,
  `is_deleted` tinyint(1) NOT NULL DEFAULT '0',
  PRIMARY KEY (`user_id`),
  KEY `index_user_id` (`user_id`) KEY_BLOCK_SIZE=10
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `moment` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `user_id` varchar(127) COLLATE utf8mb4_unicode_ci NOT NULL,
  `content` text COLLATE utf8mb4_unicode_ci,
  `gmt_created` datetime NOT NULL,
  `gmt_modified` datetime DEFAULT NULL,
  `is_deleted` tinyint(1) DEFAULT '0',
  PRIMARY KEY (`id`),
  KEY `index_user_id` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

create database test2;

CREATE TABLE `comment` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `moment_id` bigint(20) unsigned NOT NULL,
  `content` text COLLATE utf8mb4_unicode_ci,
  `gmt_created` datetime NOT NULL,
  `gmt_modified` datetime DEFAULT NULL,
  `is_deleted` tinyint(1) DEFAULT '0',
  PRIMARY KEY (`id`),
  KEY `index_moment_id` (`moment_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
1.系统环境需要Java 8 和 maven 3及以上
2.运行命令如下 (需要按顺序运行每个模块):
    cd spring-cloud-parent
    mvn clean install -DskipTests
    cd ../spring-cloud-client
    mvn clean install -DskipTests
    cd ../spring-cloud-starter
    mvn clean install -DskipTests
    
    cd spring-cloud-eureka
    mvn clean spring-boot:run
    
    cd spring-cloud-account
    mvn clean spring-boot:run
    
    cd spring-cloud-biz
    mvn clean spring-boot:run   
    
    cd spring-cloud-gateway
    mvn clean spring-boot:run
1.浏览器中打开 http://127.0.0.1:7001/index 
代码解释
1.
代码中的依赖关系见下图:
2.

3.
代码依赖关系
4.
5.spring-cloud-parent 
1.是一个空的mvn project, 包含了一些被其他项目所需要的公共的依赖。我这里创建parent，仅仅是因为我不想把很多相同的依赖在spring-cloud-eureka、spring-cloud-biz、spring-cloud-account和spring-cloud-gateway这几个项目中重复写一遍，所以，一般微服务project都集成spring-cloud-parent。嗯，懒惰是人类的第一生产力。
6.spring-cloud-starter 
1.是一个我创建的spring starter, 里面包含的是一些公共的bean，和一些公共的bean配置，比如国际化locle配置、缓存CacheService配置，messageConvertor配置等，spring-cloud-biz、spring-cloud-account和spring-cloud-gateway等微服务都需要依赖和共用这些基础配置。
7.spring-cloud-client 
1.是一个公共依赖，这里包含的是一个util类，以及多个模块需要共同使用的、和数据库表对应的Java model.
8.spring-cloud-eureka 
1.是一个服务注册和服务发现中心，需要集群化。代码里，我把spring-cloud-config动态配置项也集成在这里面了。服务发现的心跳检测时间，也从15s改到了5s，这对于快速发现节点故障很有作用。
9.spring-cloud-account 
1.是整个工程的账号服务模块。对于一个大型的系统来讲，单独的账号服务系统是很有必要解耦出来的。
10.spring-cloud-biz 
1.是实际的业务模块，包括发微博模块和评论模块。整个代码中动态使用了两个数据库，我这里对数据库的切分只是简单从业务上切分，既：微博在一个库中，评论在另外一个库中，同时写了一个@TargetDataSource注解，方便动态切换数据库。其实对于分库、分表、读写分离，代码都是类似的。
11.spring-cloud-gateway 
1.是整个系统的网关服务，所有来自端上的请求，包括app或者web页面，都需要先到网关，由网关做过处理之后再转发给其他服务。比如，网关需要处理登录状态问题，需要处理上传文件问题，需要做一些过滤，以及其他多种切面上的事情。
12.如果需要停止某个微服务，如spring-cloud-account、spring-cloud-biz或者spring-cloud-gateway, 运行命令: curl -H 'Accept:application/json' -X POST 'http://127.0.0.1:${management.port}/shutdown', 这条命令会先让这个微服务在注册中心下掉自己，然后再停掉自己。
部署
1.
服务的部署如下图:
2.

3.
部署
4.
5.需要注意的就是，整个系统只有gateway是可以被端访问到的，其他服务的所有接口，都应该只能在内网访问，这也可以让其他服务不做很强的权限验证。
6.代码里面细节很多，比如cache如何使用， GlobalCacheHelper，hystrix的自定义设置，全链路traceId的设置，信息的国际化、健康检查(去检查了cache、db等其他项，还检查了每一项的返回时间)，这里没办法展开一一讲解。有问题请直接github上开issue，或者公众号联系我：不如假如。
