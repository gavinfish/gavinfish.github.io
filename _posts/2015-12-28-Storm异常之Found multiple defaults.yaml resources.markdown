---
layout: post
title:  "Storm异常之Found multiple defaults.yaml resources"
date:   2015-12-28
categories: 大数据
tags: Storm

---

# Storm异常之RuntimeException: Found multiple defaults.yaml resources

---

## 异常

今天在运行Storm与Kafka集成的Topology时抛出如下异常：

```
java.lang.RuntimeException: Found multiple defaults.yaml resources
	at backtype.storm.utils.Utils.findAndReadConfigFile(Utils.java:106)
	at backtype.storm.utils.Utils.readDefaultConfig(Utils.java:126)
	at backtype.storm.utils.Utils.readStormConfig(Utils.java:146)
	at backtype.storm.config$read_storm_config.invoke(config.clj:66)
	at backtype.storm.testing$mk_local_storm_cluster.doInvoke(testing.clj:103)
	at clojure.lang.RestFn.invoke(RestFn.java:398)
	at backtype.storm.LocalCluster$_init.invoke(LocalCluster.clj:10)
	at backtype.storm.LocalCluster.<init>(Unknown Source)
	at TopologyMain.main(TopologyMain.java:29)
```

## 解决方案

搜索了一下依赖包，发现storm-core下面也存在一个defaults.yaml文件。修改Maven配置文件，添加scope属性并设为provided：

```xml
		<dependency>
			<groupId>org.apache.storm</groupId>
			<artifactId>storm-core</artifactId>
			<version>0.9.6</version>
			<scope>provided</scope>
		</dependency>
```

重新打包运行，异常消失。

## 知识点回忆

下面罗列一下scope的使用场景和说明：

> 注：以下内容摘录自acooly的博客文章 [MAVEN Scope使用](http://acooly.iteye.com/blog/1788890)

**1.compile**
编译范围，默认scope，在工程环境的classpath（编译环境）和打包（如果是WAR包，会包含在WAR包中）时候都有效。
 
**2.provided**
容器或JDK已提供范围，表示该依赖包已经由目标容器（如tomcat）和JDK提供，只在编译的classpath中加载和使用，打包的时候不会包含在目标包中。最常见的是j2ee规范相关的servlet-api和jsp-api等jar包，一般由servlet容器提供，无需在打包到war包中，如果不配置为provided，把这些包打包到工程war包中，在tomcat6以上版本会出现冲突无法正常运行程序（版本不符的情况）。
 
**3.runtime**
一般是运行和测试环境使用，编译时候不用加入classpath，打包时候会打包到目标包中。一般是通过动态加载或接口反射加载的情况比较多。也就是说程序只使用了接口，具体的时候可能有多个，运行时通过配置文件或jar包扫描动态加载的情况。典型的包括：JDBC驱动等。
 
**4.test**
测试范围，一般是单元测试场景使用，在编译环境加入classpath，但打包时不会加入，如junit等。
 
**5.system**
系统范围，与provided类似，只是标记为该scope的依赖包需要明确指定基于文件系统的jar包路径。因为需要通过systemPath指定本地jar文件路径，所以该scope是不推荐的。如果是基于组织的，一般会建立本地镜像，会把本地的或组织的基础组件加入本地镜像管理，避过使用该scope的情况。
 
**实践：**
provided是没有传递性的，也就是说，如果你依赖的某个jar包，它的某个jar的范围是provided，那么该jar不会在你的工程中依靠jar依赖传递加入到你的工程中。
provided具有继承性，上面的情况，如果需要统一配置一个组织的通用的provided依赖，可以使用parent，然后在所有工程中继承。






