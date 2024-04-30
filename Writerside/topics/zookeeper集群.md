# Zookeeper集群

<note>
    <p>
        本文中构建的是单机集群，仅作为开发和测试使用 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </p>
</note>


## 准备工作

1. 拉取zookeeper镜像

<code-block lang="shell">
docker pull zookeeper
docker tag docker.io/zookeeper zookeeper
docker rmi docker.io/zookeeper
</code-block>

2. 安装docker-compose工具

<code-block lang="shell">
# 升级 pip
pip3 install --upgrade pip
# 指定 docker-compose 版本安装
pip install docker-compose==1.22
# 验证是否安装成功，有返回值，说明安装成功
docker-compose -v
</code-block>

3. 创建相关文件夹

<code-block lang="shell">
mkdir -p /data/docker-compose/zookeeper
mkdir -p /data/docker-data/zookeeper
</code-block>

## docker-compose编排zookeeper集群

1. 创建docker-compose.yml

<code-block lang="shell">
cd /data/docker-compose/zookeeper
vi docker-compose.yml
</code-block>

```yaml
version: '3.6'
services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    container_name: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
    volumes:
      - /data/docker-data/zookeeper/zoo1/data:/data
      - /data/docker-data/zookeeper/zoo1/datalog:/datalog

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    container_name: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
    volumes:
      - /data/docker-data/zookeeper/zoo2/data:/data
      - /data/docker-data/zookeeper/zoo2/datalog:/datalog

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    container_name: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
    volumes:
      - /data/docker-data/zookeeper/zoo3/data:/data
      - /data/docker-data/zookeeper/zoo3/datalog:/datalog
```
{collapsible="true" collapsed-title="docker-compose.yml"}

2. 执行构建集群

<code-block lang="shell">
cd /data/docker-compose/zookeeper
docker-compose up -d
</code-block>

3. 进入容器验证集群是否构建成功

<code-block lang="shell">
docker exec -it zoo1 bash
zkServer.sh status
</code-block>

4. 返回下面信息，说明集群搭建成功:

<code-block lang="shell">
root@zoo1:/# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower
</code-block>

可以看到，zoo1是follower角色，这是三台zookeeper内部选举的结果，无需我们干预。

到这里，zookeeper集群就搭建好了。