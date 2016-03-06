---
layout: post
title:  "Storm新特性之Flux"
date:   2016-03-05
categories: 大数据
tags: Storm

---

# Storm新特性之Flux

---

Flux是Storm版本0.10.0中的新组件，主要目的是为了方便拓扑的开发与部署。原先在开发Storm拓扑的时候整个拓扑的结构都是硬编码写在代码中的，当要对其进行修改时，需要修改代码并重新编译和打包，这是一件繁琐和痛苦的事情，Flux解决了这一问题。

## 特性

下面是Flux提供的所有的特性：

- 容易配置和部署拓扑（包括Storm和Trident）
- 支持变更已存在的拓扑
- 通过YAML文件来定义Spouts和Bolts，甚至可以支持Storm的其他组件，如storm-kafka/storm-hdfs/storm-hbase等
- 容易支持多语言协议组件
- 方便在不同环境中切换

## 使用

想要用Flux最简单的方法就是添加Maven依赖，然后打包成一个胖jar文件。依赖配置如下：

```xml
<!-- include Flux and user dependencies in the shaded jar -->
<dependencies>
    <!-- Flux include -->
    <dependency>
        <groupId>org.apache.storm</groupId>
        <artifactId>flux-core</artifactId>
        <version>${storm.version}</version>
    </dependency>

    <!-- add user dependencies here... -->

</dependencies>
<!-- create a fat jar that includes all dependencies -->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>1.4</version>
            <configuration>
                <createDependencyReducedPom>true</createDependencyReducedPom>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                            <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>org.apache.storm.flux.Flux</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

接下来是YAML文件的定义，一个拓扑的定义需要包括以下的部分：

1. 拓扑名
2. 拓扑组件的列表
3. spouts、bolts、stream，或者是一个可以提供`org.apache.storm.generated.StormTopology`实例的JVM类。

下面是YAML文件实例：

```yaml
name: "yaml-topology"
config:
  topology.workers: 1

# spout definitions
spouts:
  - id: "spout-1"
    className: "org.apache.storm.testing.TestWordSpout"
    parallelism: 1

# bolt definitions
bolts:
  - id: "bolt-1"
    className: "org.apache.storm.testing.TestWordCounter"
    parallelism: 1
  - id: "bolt-2"
    className: "org.apache.storm.flux.wrappers.bolts.LogInfoBolt"
    parallelism: 1

#stream definitions
streams:
  - name: "spout-1 --> bolt-1" # name isn't used (placeholder for logging, UI, etc.)
    from: "spout-1"
    to: "bolt-1"
    grouping:
      type: FIELDS
      args: ["word"]

  - name: "bolt-1 --> bolt2"
    from: "bolt-1"
    to: "bolt-2"
    grouping:
      type: SHUFFLE

```

在有了jar文件和YAML文件后就可以通过以下的命令运行Flux拓扑了，其中`myTopology-0.1.0-SNAPSHOT.jar`是打包后的jar文件，`org.apache.storm.flux.Flux`是Flux的入口类，`--local`表示是在本地运行拓扑，`my_config.yaml`使YAML配置文件。

```
storm jar myTopology-0.1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local my_config.yaml
```

## 其他特性详解

### 不同环境切换

在不同的环境中运行拓扑需要不一样的配置，如开发环境和生产环境，这些环境中切换一般不会改变拓扑的结构，只是要修改主机、端口号和并行度等。如果用两份不一样的YAML文件来进行会产生不必要的重复，Flux可以通过.properites文件来加载不同的环境变量。只需要添加`--filter`参数即可：

```
torm jar myTopology-0.1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local my_config.yaml --filter dev.properties
```

以YAML文件中的Kafka主机为例，YAML文件修改如下：

```yaml
- id: "zkHosts"
    className: "org.apache.storm.kafka.ZkHosts"
    constructorArgs:
      - "${kafka.zookeeper.hosts}"
```

而dev.properties问价如下：

```
kafka.zookeeper.hosts: localhost:2181
```

> 注：YAML文件中也可以解析系统环境变量${ENV-VARIABLE}

### 多语言协议的支持

多语言特性的支持比较简单，只需要修改YAML文件中构造参数，如下面是一个由Python写成的bolts：

```yaml
bolts:
  - id: "splitsentence"
    className: "org.apache.storm.flux.bolts.GenericShellBolt"
    constructorArgs:
      # command line
      - ["python", "splitsentence.py"]
      # output fields
      - ["word"]
    parallelism: 1
```

##　展望
Flux虽然可以加方便拓扑的修改与部署，但这仍然不支持动态的修改拓扑结构，在修改拓扑时仍要中断并重启。不过现在在开发中的几个特性有望改善这个情况。

- 本文由 DRFish（http://www.drfish.me/）原创，转载请写明原链接，谢谢。

参考内容： [Flux github](https://github.com/apache/storm/blob/a4f9f8bc5b4ca85de487a0a868e519ddcb94e852/external/flux/README.md)

