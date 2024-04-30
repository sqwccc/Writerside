# KAFKA集群的搭建

<note>
    <p>
        本文中构建的是单机集群，仅作为开发和测试使用 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </p>
</note>


## 准备工作

拉取kafka镜像
```Shell
docker pull wurstmeister/kafka
docker tag docker.io/wurstmeister/kafka kafka
docker rmi docker.io/wurstmeister/kafka
```

拉取kafka可视化管理工具镜像
```Shell
docker pull sheepkiller/kafka-manager
docker tag docker.io/sheepkiller/kafka-manager kafka-manager
docker rmi docker.io/sheepkiller/kafka-manager
```

安装docker-compose工具
```Shell
# 升级 pip
pip3 install --upgrade pip

# 指定 docker-compose 版本安装
pip install docker-compose==1.22

# 验证是否安装成功，有返回值，说明安装成功
docker-compose -v
```

创建相关文件夹
```Shell
mkdir -p /data/docker-compose/kafka
mkdir -p /data/docker-data/kafka
```

创建网络，用于kafka和zookeeper共享一个网络段
```Shell
docker network create --driver bridge zookeeper_kafka_net
```

构建zookeeper集群

kafka集群需要用到zookeeper集群，因此需要先构建zookeeper集群，请查看文章[zookeeper集群构建](zookeeper集群.md)

## 使用docker-compose编排kafka集群

1. 创建docker-compose.yml

```Shell
cd /data/docker-compose/kafka
vi docker-compose.yml
```

```yaml
version: '3'

services:
  kafka1:
    image: kafka
    restart: always
    container_name: kafka1
    hostname: kafka1
    ports:
      - 9091:9092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ADVERTISED_HOST_NAME: kafka1
      KAFKA_ADVERTISED_PORT: 9091
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.78.200:9091
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
    volumes:
      - /data/docker-data/kafka1/docker.sock:/var/run/docker.sock
      - /data/docker-data/kafka1/data:/kafka
    external_links:
      - zoo1
      - zoo2
      - zoo3

  kafka2:
    image: kafka
    restart: always
    container_name: kafka2
    hostname: kafka2
    ports:
      - 9092:9092
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ADVERTISED_HOST_NAME: kafka2
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.78.200:9092
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
    volumes:
      - /data/docker-data/kafka2/docker.sock:/var/run/docker.sock
      - /data/docker-data/kafka2/data:/kafka
    external_links:
      - zoo1
      - zoo2
      - zoo3

  kafka3:
    image: kafka
    restart: always
    container_name: kafka3
    hostname: kafka3
    ports:
      - 9093:9092
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ADVERTISED_HOST_NAME: kafka3
      KAFKA_ADVERTISED_PORT: 9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.78.200:9093
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
    volumes:
      - /data/docker-data/kafka3/docker.sock:/var/run/docker.sock
      - /data/docker-data/kafka3/data:/kafka
    external_links:
      - zoo1
      - zoo2
      - zoo3

  kafka-manager:
    image: kafka-manager
    restart: always
    container_name: kafka-manager
    hostname: kafka-manager
    ports:
      - 9010:9000
    links:
      - kafka1
      - kafka2
      - kafka3
    external_links:
      - zoo1
      - zoo2
      - zoo3
    environment:
      ZK_HOSTS: zoo1:2181,zoo2:2181,zoo3:2181
```
{collapsible="true" collapsed-title="docker-compose.yml"}

<note>
    <p>
        links是引入当前docker-compose内部的service，external_links 引入的是当前docker-compose外部的service &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </p>
</note>
<warning>
    <p>
        将KAFKA_ADVERTISED_LISTENERS后的ip替换为你服务器的ip &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </p>
</warning>

2. 执行构建

```Shell
docker-compose up -d
```

3. 将所有kafka和zookeeper加入一个网络

```Shell
docker network connect zookeeper_kafka_net zoo1
docker network connect zookeeper_kafka_net zoo2
docker network connect zookeeper_kafka_net zoo3
docker network connect zookeeper_kafka_net kafka1
docker network connect zookeeper_kafka_net kafka2
docker network connect zookeeper_kafka_net kafka3
docker network connect zookeeper_kafka_net kafka-manager
```

之所以这样指定网络而不是在编排文件中指定网络为一个网络段的原因是因为考虑到后面可能kafka和zookeeper分布在6台机器上， 到时使用docker swarm构建docker 集群，网络处理就很方便，不用修改编排文件。

4. 重启这组服务
   
因为为每台容器都新增了一个网络，如果不重启，容器用的还是之前的网络，就会导致kafka和kafka-manager是ping不通zookeeper的三个容器。

## 验证结果

浏览器访问kafka-manager管理界面，**YOUR_ADDRESS**:9010

看到如下界面说明连接成功。

![kafka_manager](kafka_manager.png)

## 使用Idea的kafka插件提升开发效率


<p>配置你的kafka插件: 使用 <shortcut>,</shortcut> 来分割节点之间的地址</p>

![kafka_settings](idea_kafka.png)


功能简介：
<tabs>
    <tab title="主题管理">
        <img src="topic_kafka.png" alt="kafka"/>
    </tab>
    <tab title="模拟消费者生产者">
        <img src="kafka_cos_pro.png" alt="kafka"/>
    </tab>
    <tab title="分区和配置信息管理">
        <img src="kafka_info.png" alt="kafka"/>
    </tab>
</tabs>
