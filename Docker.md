# 47.96.100.1661、安装Docker

## 1.1 Ubuntu 14.04/16.04 (使用apt-get进行安装)

```powershell
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]
```

## 1.2 CentOS 7 (使用yum进行安装)

```powershell
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start
# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，您可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ee.repo
#   将[docker-ce-test]下方的enabled=0修改为enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```

# 2、 安装校验

```powershell
root@iZbp12adskpuoxodbkqzjfZ:$ docker version
Client:
 Version:      17.03.0-ce
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 07:52:04 2017
 OS/Arch:      linux/amd64
Server:
 Version:      17.03.0-ce
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 07:52:04 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

# 3、启动 Docker CE

```powershell
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

Ubuntu 14.04 请使用以下命令启动：

```shell
$ sudo service docker start
```

# 4、镜像加速

国内从 Docker Hub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务，例如：

- [Docker 官方提供的中国 registry mirror](https://docs.docker.com/registry/recipes/mirror/#use-case-the-china-registry-mirror)
- [阿里云加速器](https://cr.console.aliyun.com/#/accelerator)
- [DaoCloud 加速器](https://www.daocloud.io/mirror#accelerator-doc)

对于使用 systemd（Ubuntu 16.04+、Debian 8+、CentOS 7） 的系统，请在` /etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

```json
{
  "registry-mirrors": [
    "https://7jlsf15h.mirror.aliyuncs.com"
  ]
}
```

注意，一定要保证该文件符合 json 规范，否则 Docker 将不能启动。

之后重新启动服务。

```shell
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

# 5、Docker Compose 安装与卸载

Compose 支持 Linux、macOS、Windows 10 三大平台。

Compose 可以通过 Python 的包管理工具 pip 进行安装，也可以直接下载编译好的二进制文件使用，甚至能够直接在 Docker 容器中运行。

前两种方式是传统方式，适合本地环境下安装使用；最后一种方式则不破坏系统环境，更适合云计算场景。

Docker for Mac 、Docker for Windows 自带 docker-compose 二进制文件，安装 Docker 之后可以直接使用。

```shell
$ docker-compose --version

docker-compose version 1.17.1, build 6d101fb
```

Linux 系统请使用以下介绍的方法安装。

## 5.1 二进制包

在 Linux 上的也安装十分简单，从 [官方 GitHub Release](https://github.com/docker/compose/releases) 处直接下载编译好的二进制文件即可。

例如，在 Linux 64 位系统上直接下载对应的二进制包。

```she
$ sudo curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

## 5.2 PIP 安装

*注：* `x86_64` 架构的 Linux 建议按照上边的方法下载二进制包进行安装，如果您计算机的架构是 `ARM` (例如，树莓派)，再使用 `pip` 安装。

这种方式是将 Compose 当作一个 Python 应用来从 pip 源中安装。

执行安装命令：

```she
$ sudo apt-get install python-pip
$ sudo pip install -U docker-compose
```

可以看到类似如下输出，说明安装成功。

```shell
Collecting docker-compose
  Downloading docker-compose-1.17.1.tar.gz (149kB): 149kB downloaded
...
Successfully installed docker-compose cached-property requests texttable websocket-client docker-py dockerpty six enum34 backports.ssl-match-hostname ipaddress
```

# 6、安装zookeeper

Zookeeper 部署有三种方式，单机模式、集群模式、伪集群模式，以下采用 Docker 的方式部署

**注意：** 集群为大于等于3个奇数，如 3、5、7,不宜太多，集群机器多了选举和数据同步耗时长，不稳定。

```
docker pull zookeeper:3.5.4-beta
docker run --name zookeeper -p 2181:2181 --restart always -d zookeeper:3.5.4-beta
```

## 6.1 单机模式

#### docker-compose.yml

```she
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    ports:
        - 2181:2181
    environment:
        ZOO_MY_ID: 1
        ZOO_SERVERS: server.1=121.43.162.28:2888:3888
```

#### 验证是否安装成功

- 以交互的方式进入容器

  ```
  docker exec -it zookeeper_zoo1_1 /bin/bash
  ```

- 使用客户端连接到服务端

  ```
  bash-4.3# ./bin/zkCli.sh -server 192.168.75.130:2181
  Connecting to 192.168.75.130:2181
  2017-11-09 07:45:58,365 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
  2017-11-09 07:45:58,374 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=zoo1
  2017-11-09 07:45:58,374 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_131
  2017-11-09 07:45:58,380 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
  2017-11-09 07:45:58,381 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-1.8-openjdk/jre
  2017-11-09 07:45:58,381 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/zookeeper-3.4.10/bin/../build/classes:/zookeeper-3.4.10/bin/../build/lib/*.jar:/zookeeper-3.4.10/bin/../lib/slf4j-log4j12-1.6.1.jar:/zookeeper-3.4.10/bin/../lib/slf4j-api-1.6.1.jar:/zookeeper-3.4.10/bin/../lib/netty-3.10.5.Final.jar:/zookeeper-3.4.10/bin/../lib/log4j-1.2.16.jar:/zookeeper-3.4.10/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.10/bin/../zookeeper-3.4.10.jar:/zookeeper-3.4.10/bin/../src/java/lib/*.jar:/conf:
  2017-11-09 07:45:58,381 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64/server:/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64:/usr/lib/jvm/java-1.8-openjdk/jre/../lib/amd64:/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
  2017-11-09 07:45:58,381 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
  2017-11-09 07:45:58,381 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
  2017-11-09 07:45:58,381 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
  2017-11-09 07:45:58,382 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
  2017-11-09 07:45:58,382 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=4.4.0-98-generic
  2017-11-09 07:45:58,386 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
  2017-11-09 07:45:58,386 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
  2017-11-09 07:45:58,386 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/zookeeper-3.4.10
  2017-11-09 07:45:58,389 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=192.168.75.130:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@3eb07fd3
  2017-11-09 07:45:58,428 [myid:] - INFO  [main-SendThread(192.168.75.130:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server 192.168.75.130/192.168.75.130:2181. Will not attempt to authenticate using SASL (unknown error)
  Welcome to ZooKeeper!
  JLine support is enabled
  2017-11-09 07:45:58,529 [myid:] - INFO  [main-SendThread(192.168.75.130:2181):ClientCnxn$SendThread@876] - Socket connection established to 192.168.75.130/192.168.75.130:2181, initiating session
  [zk: 192.168.75.130:2181(CONNECTING) 0] 2017-11-09 07:45:58,573 [myid:] - INFO  [main-SendThread(192.168.75.130:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server 192.168.75.130/192.168.75.130:2181, sessionid = 0x15f9fbc12ec0000, negotiated timeout = 30000
  
  WATCHER::
  
  WatchedEvent state:SyncConnected type:None path:null
  ```


- 使用服务端工具检查服务器状态

  ```
  bash-4.3# ./bin/zkServer.sh status
  ZooKeeper JMX enabled by default
  Using config: /conf/zoo.cfg
  Mode: standalone
  ```




## 6.2 真实集群模式（三台服务器）

准备 3 台 Ubuntu Server 系统，并分别配置 Zookeeper

#### 第一台主机

##### docker-compose.yml

```
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    network_mode: host
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=47.96.100.166:2888:3888 server.3=118.24.136.237:2888:3888
```

ps：当前服务的ip不能用真实ip，必须用`0.0.0.0`，不然启动会报错，集群失败（排查好久的教训）

##### 验证测试

```
root@UbuntuBase:/usr/local/docker/zookeeper# docker exec -it zookeeper_zoo1_1 /bin/bash
bash-4.3# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Mode: leader
```

##### 查看集群的运行状态

使用`nc`命令。通过`nc`命令连接到指定的`ZooKeeper`服务器, 然后发送`stat`可以查看服务的状态。

```
$ echo stat | nc 121.43.162.28 2181

Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /192.168.1.102:48174[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000002
Mode: follower
Node count: 4
```



#### 第二台主机

##### docker-compose.yml

```
version: '3.1'

services:
  zoo2:
    image: zookeeper
    restart: always
    network_mode: host
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=121.43.162.28:2888:3888 server.2=0.0.0.0:2888:3888 server.3=118.24.136.237:2888:3888
```

ps：当前服务的ip不能用真实ip，必须用`0.0.0.0`，不然启动会报错，集群失败（排查好久的教训）

##### 验证测试

```
root@UbuntuBase:/usr/local/docker/zookeeper# docker exec -it zookeeper_zoo2_1 /bin/bash
bash-4.3# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Mode: follower
```

#### 第三台主机

##### docker-compose.yml

```
version: '3.1'

services:
  zoo3:
    image: zookeeper
    restart: always
    network_mode: host
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=121.43.162.28:2888:3888 server.2=47.96.100.166:2888:3888 server.3=0.0.0.0:2888:3888
```

ps：当前服务的ip不能用真实ip，必须用`0.0.0.0`，不然启动会报错，集群失败（排查好久的教训）

##### 验证测试

```
root@UbuntuBase:/usr/local/docker/zookeeper# docker exec -it zookeeper_zoo3_1 /bin/bash
bash-4.3# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Mode: follower
```

## 6.3 、伪集群模式

#### docker-compose.yml

```
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=0.0.0.0:2888:3888 server.3=zoo3:2888:3888

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=0.0.0.0:2888:3888
```

#### 验证是否安装成功

- 分别以交互方式进入容器查看

```
docker exec -it zookeeper_zoo1_1 /bin/bash
docker exec -it zookeeper_zoo2_1 /bin/bash
docker exec -it zookeeper_zoo3_1 /bin/bash
```

- 使用服务端工具检查服务器状态

```
root@UbuntuBase:/usr/local/docker/zookeeper# docker exec -it zookeeper_zoo1_1 /bin/bash
bash-4.3# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Mode: follower
root@UbuntuBase:/usr/local/docker/zookeeper# docker exec -it zookeeper_zoo2_1 /bin/bash
bash-4.3# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Mode: follower
root@UbuntuBase:/usr/local/docker/zookeeper# docker exec -it zookeeper_zoo3_1 /bin/bash
bash-4.3# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Mode: leader
```

从上面的验证结果可以看出：zoo1 为跟随者，zoo2 为跟随者，zoo3 为领导者

#  7、 安装rabbitmq

```she
docker pull rabbitmq:3-management
docker run --name rabbitmq -p 5672:5672 -p 15672:15672 -d rabbitmq:3-management
```



# 8、安装mysql

解决乱码，文件`/data/mysql/conf/my.cnf`

```
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
collation-server=utf8_general_ci
character-set-server=utf8
init-connect='SET NAMES utf8'
```

```shel
docker pull mysql:5.7 
docker run -d -p 3306:3306 -v /data/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=passw0rd --name mysql mysql:5.7 
```



# 9、docker安装redis 

## 9.1 单机

```she
docker run --name redis-6380 -p 6380:6379 -d redis:4.0 --requirepass "passw0rd"
```

## 9.2 主从

```
docker run --name redis-6383 -p 6383:6379 -d redis:4.0 --requirepass "passw0rd"
 
docker run --name redis-6384 -p 6384:6379 -d redis:4.0 --slaveof 121.43.162.28 6383 \
--requirepass "passw0rd" --masterauth "passw0rd"
 
docker run --name redis-6385 -p 6385:6379 -d redis:4.0 --slaveof 121.43.162.28 6383 \
--requirepass "passw0rd" --masterauth "passw0rd"
```

ps:由于没采用挂载文件的方式启动，一些配置不妨通过命令的方式带进去，如下

```
-- appendonly yes
```

## 9.3 挂载配置文件的方式启动

配置文件 `/usr/local/redis/redis.conf`

```
requirepass passw0rd

## 以下为rdb配置
#dbfilename：持久化数据存储在本地的文件
dbfilename dump.rdb

#dir：持久化数据存储在本地的路径，如果是在/redis/redis-3.0.6/src下启动的redis-cli，则数据会存储在当前src目录下
dir ./

##snapshot触发的时机，save <seconds> <changes>  
##如下为900秒后，至少有一个变更操作，才会snapshot  
##对于此值的设置，需要谨慎，评估系统的变更操作密集程度  
##可以通过“save “””来关闭snapshot功能  
#save时间，以下分别表示更改了1个key时间隔900s进行持久化存储；更改了10个key300s进行存储；更改10000个key60s进行存储。
save 900 1
save 300 10
save 60 10000

##当snapshot时出现错误无法继续时，是否阻塞客户端“变更操作”，“错误”可能因为磁盘已满/磁盘故障/OS级别异常等  
stop-writes-on-bgsave-error yes 

##是否启用rdb文件压缩，默认为“yes”，压缩往往意味着“额外的cpu消耗”，同时也意味这较小的文件尺寸以及较短的网络传输时间  
rdbcompression yes

##以下为aof配置
##此选项为aof功能的开关，默认为“no”，可以通过“yes”来开启aof功能  
##只有在“yes”下，aof重写/文件同步等特性才会生效  
appendonly yes  

##指定aof文件名称  
appendfilename appendonly.aof  

##指定aof操作中文件同步策略，有三个合法值：always everysec no,默认为everysec  
appendfsync everysec  

##在aof-rewrite期间，appendfsync是否暂缓文件同步，"no"表示“不暂缓”，“yes”表示“暂缓”，默认为“no”
no-appendfsync-on-rewrite no  

##aof文件rewrite触发的最小文件尺寸(mb,gb),只有大于此aof文件大于此尺寸是才会触发rewrite，默认“64mb”，建议“512mb”  
auto-aof-rewrite-min-size 64mb  

##相对于“上一次”rewrite，本次rewrite触发时aof文件应该增长的百分比。  
##每一次rewrite之后，redis都会记录下此时“新aof”文件的大小(例如A)，那么当aof文件增长到A*(1 + p)之后  
##触发下一次rewrite，每一次aof记录的添加，都会检测当前aof文件的尺寸。  
auto-aof-rewrite-percentage 100 

#主节点密码
masterauth passw0rd
```

启动方式

```
docker run --name redis-6379 -p 6379:6379  \
-v /usr/local/redis/redis.conf:/etc/redis/redis.conf \
-d redis:4.0 redis-server /etc/redis/redis.conf
```



# 10、docker compose 安装redis

## 10.1 redis主从安装

```
mkdir -p /usr/local/master-slave
cd /usr/local/master-slave && touch docker-compose.yml
```

文件内容

```
version: "3.1"
services:

  redis-6383:
    image: redis:4.0
    ports: 
      - "6383:6379"
    restart: always
    command: [
      '--requirepass "passw0rd"',
      '--masterauth "passw0rd"',
      '--maxmemory 512mb',
      '--maxmemory-policy volatile-ttl',
      '--save ""',
    ]

  redis-6384:
    image: redis:4.0
    ports: 
      - "6384:6379"
    restart: always
    depends_on: 
      - redis-6383
    command: [
      '--requirepass "passw0rd"',
      '--masterauth "passw0rd"',
      '--maxmemory 512mb',
      '--maxmemory-policy volatile-ttl',
      '--slaveof 121.43.162.28 6383',
      '--save ""',
    ]

  redis-6385:
    image: redis:4.0
    ports: 
      - "6385:6379"
    restart: always
    depends_on: 
      - redis-6383
    command: [
      '--requirepass "passw0rd"',
      '--masterauth "passw0rd"',
      '--maxmemory 512mb',
      '--maxmemory-policy volatile-ttl',
      '--slaveof 121.43.162.28 6383',
      '--save ""',
    ]   
```

```
cd /usr/local/master-slave
#启动
docker-compose up -d
#停止
docker-compose stop
#删除
docker-compose rm
```



## 10.2 redis哨兵集群安装

#### 第一台主机

```
mkdir -p /usr/local/docker/redis-sentinel
cd /usr/local/docker/redis-sentinel 
touch sentinel.conf && touch docker-compose.yml
```

```
# 守护进程模式(千万别加这个)
#daemonize yes
# 当前Sentinel服务运行的端口
port 26379

# Sentinel服务运行时使用的临时文件夹
dir /data

# Sentinel去监视一个名为mymaster的主redis实例，这个主实例的IP地址为redis-master，端口号为6379，而将这个主实例判断为失效至少需要2个Sentinel进程的同意，只要同意Sentinel的数量不达标，自动failover就不会执行
sentinel monitor mymaster 121.43.162.28 6379 2

# 指定了Sentinel认为Redis实例已经失效所需的毫秒数。当实例超过该时间没有返回PING，或者直接返回错误，那么Sentinel将这个实例标记为主观下线。只有一个 Sentinel进程将实例标记为主观下线并不一定会引起实例的自动故障迁移：只有在足够数量的Sentinel都将一个实例标记为主观下线之后，实例才会被标记为客观下线，这时自动故障迁移才会执行
sentinel down-after-milliseconds mymaster 30000

# 指定了在执行故障转移时，最多可以有多少个从Redis实例在同步新的主实例，在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长
sentinel parallel-syncs mymaster 1

# 如果在该时间（ms）内未能完成failover操作，则认为该failover失败
sentinel failover-timeout mymaster 180000

# 设置主服务密码
sentinel auth-pass mymaster passw0rd
```

但这里面有个坑，也是困扰我一阵的，因为哨兵没指定IP和端口号，会自动检测通常的本地地址，本地哨兵之间通信都是用默认的IP地址和端口号(单机部署，都是默认的局域网IP)，这本身也没问题，但一旦客户端连接的时候，就算你配置了你认为的三个哨兵，并且都是你暴露出去的外网地址和端口号，但第一次能连上，连接完毕后，哨兵开始通信，却用到的是局域网IP，而这时候，你是连接不上的，所以我们需要为每个哨兵声明指定的IP地址。

`sentinel.conf`追加

```
#声明IP地址和端口号
sentinel announce-ip 121.43.162.28
sentinel announce-port 26379
```

`docker-compose.yml`内容如下

```
version: "3.1"
services:

  redis-6379:
    image: redis:4.0
    ports: 
      - "6379:6379"
    restart: always
    command: [
      '--requirepass "passw0rd"',
      '--masterauth "passw0rd"',
      '--maxmemory 512mb',
      '--maxmemory-policy volatile-ttl',
      '--save ""',
    ]

  redis-sentinel:
    image: redis:4.0
    restart: always    
    ports: 
      - "26379:26379"
    volumes: 
      - "/usr/local/docker/redis-sentinel/sentinel.conf:/usr/local/etc/redis/sentinel.conf"
    privileged: true
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    depends_on: 
      - redis-6379
```

```
cd /usr/local/docker/master-sentinel
#启动
docker-compose up -d
#停止
docker-compose stop
#删除
docker-compose rm
```

验证节点

```
redis-cli -c -p 6379
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=47.96.100.166,port=6379,state=online,offset=17443,lag=1
slave1:ip=118.24.136.237,port=6379,state=online,offset=17443,lag=1
master_replid:954733f7a22868fc4a120ca6d12f55c98c59e7a8
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:17584
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:17584
```

验证哨兵

```
redis-cli -c -p 26379
info sentinel
```



#### 第二台主机

```
mkdir -p /usr/local/docker/redis-sentinel
cd /usr/local/docker/redis-sentinel 
touch sentinel.conf && touch docker-compose.yml
```

```
# 守护进程模式(千万别加这个)
#daemonize yes
# 当前Sentinel服务运行的端口
port 26379

# Sentinel服务运行时使用的临时文件夹
dir /data

# Sentinel去监视一个名为mymaster的主redis实例，这个主实例的IP地址为redis-master，端口号为6379，而将这个主实例判断为失效至少需要2个Sentinel进程的同意，只要同意Sentinel的数量不达标，自动failover就不会执行
sentinel monitor mymaster 121.43.162.28 6379 2

# 指定了Sentinel认为Redis实例已经失效所需的毫秒数。当实例超过该时间没有返回PING，或者直接返回错误，那么Sentinel将这个实例标记为主观下线。只有一个 Sentinel进程将实例标记为主观下线并不一定会引起实例的自动故障迁移：只有在足够数量的Sentinel都将一个实例标记为主观下线之后，实例才会被标记为客观下线，这时自动故障迁移才会执行
sentinel down-after-milliseconds mymaster 30000

# 指定了在执行故障转移时，最多可以有多少个从Redis实例在同步新的主实例，在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长
sentinel parallel-syncs mymaster 1

# 如果在该时间（ms）内未能完成failover操作，则认为该failover失败
sentinel failover-timeout mymaster 180000

# 设置主服务密码
sentinel auth-pass mymaster passw0rd
```

但这里面有个坑，也是困扰我一阵的，因为哨兵没指定IP和端口号，会自动检测通常的本地地址，本地哨兵之间通信都是用默认的IP地址和端口号(单机部署，都是默认的局域网IP)，这本身也没问题，但一旦客户端连接的时候，就算你配置了你认为的三个哨兵，并且都是你暴露出去的外网地址和端口号，但第一次能连上，连接完毕后，哨兵开始通信，却用到的是局域网IP，而这时候，你是连接不上的，所以我们需要为每个哨兵声明指定的IP地址。

`sentinel.conf`追加

```
#声明IP地址和端口号
sentinel announce-ip 47.96.100.166
sentinel announce-port 26379
```

`docker-compose.yml`内容如下

```
version: "3.1"
services:

  redis-6379:
    image: redis:4.0
    ports: 
      - "6379:6379"
    restart: always
    command: [
      '--requirepass "passw0rd"',
      '--masterauth "passw0rd"',
      '--maxmemory 512mb',
      '--maxmemory-policy volatile-ttl',
      '--slaveof 121.43.162.28 6379',
      '--save ""',
    ]

  redis-sentinel:
    image: redis:4.0
    restart: always    
    ports: 
      - "26379:26379"
    volumes: 
      - "/usr/local/docker/redis-sentinel/sentinel.conf:/usr/local/etc/redis/sentinel.conf"
    privileged: true
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    depends_on: 
      - redis-6379
```

```
cd /usr/local/docker/master-sentinel
#启动
docker-compose up -d
#停止
docker-compose stop
#删除
docker-compose rm
```

 验证节点

```
redis-cli -c -p 6379
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:121.43.162.28
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:27688
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:954733f7a22868fc4a120ca6d12f55c98c59e7a8
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:27688
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:27688
```



#### 第三台主机

```
mkdir -p /usr/local/docker/redis-sentinel
cd /usr/local/docker/redis-sentinel 
touch sentinel.conf && touch docker-compose.yml
```

```
# 守护进程模式(千万别加这个)
#daemonize yes
# 当前Sentinel服务运行的端口
port 26379

# Sentinel服务运行时使用的临时文件夹
dir /data

# Sentinel去监视一个名为mymaster的主redis实例，这个主实例的IP地址为redis-master，端口号为6379，而将这个主实例判断为失效至少需要2个Sentinel进程的同意，只要同意Sentinel的数量不达标，自动failover就不会执行
sentinel monitor mymaster 121.43.162.28 6379 2

# 指定了Sentinel认为Redis实例已经失效所需的毫秒数。当实例超过该时间没有返回PING，或者直接返回错误，那么Sentinel将这个实例标记为主观下线。只有一个 Sentinel进程将实例标记为主观下线并不一定会引起实例的自动故障迁移：只有在足够数量的Sentinel都将一个实例标记为主观下线之后，实例才会被标记为客观下线，这时自动故障迁移才会执行
sentinel down-after-milliseconds mymaster 30000

# 指定了在执行故障转移时，最多可以有多少个从Redis实例在同步新的主实例，在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长
sentinel parallel-syncs mymaster 1

# 如果在该时间（ms）内未能完成failover操作，则认为该failover失败
sentinel failover-timeout mymaster 180000

# 设置主服务密码
sentinel auth-pass mymaster passw0rd
```

但这里面有个坑，也是困扰我一阵的，因为哨兵没指定IP和端口号，会自动检测通常的本地地址，本地哨兵之间通信都是用默认的IP地址和端口号(单机部署，都是默认的局域网IP)，这本身也没问题，但一旦客户端连接的时候，就算你配置了你认为的三个哨兵，并且都是你暴露出去的外网地址和端口号，但第一次能连上，连接完毕后，哨兵开始通信，却用到的是局域网IP，而这时候，你是连接不上的，所以我们需要为每个哨兵声明指定的IP地址。

`sentinel.conf`追加

```
#声明IP地址和端口号
sentinel announce-ip 118.24.136.237
sentinel announce-port 26379
```

`docker-compose.yml`内容如下

```
version: "3.1"
services:

  redis-6379:
    image: redis:4.0
    ports: 
      - "6379:6379"
    restart: always
    command: [
      '--requirepass "passw0rd"',
      '--masterauth "passw0rd"',
      '--maxmemory 512mb',
      '--maxmemory-policy volatile-ttl',
      '--slaveof 121.43.162.28 6379',
      '--save ""',
    ]

  redis-sentinel:
    image: redis:4.0
    restart: always    
    ports: 
      - "26379:26379"
    volumes: 
      - "/usr/local/docker/redis-sentinel/sentinel.conf:/usr/local/etc/redis/sentinel.conf"
    privileged: true
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    depends_on: 
      - redis-6379
```

```
cd /usr/local/docker/master-sentinel
#启动
docker-compose up -d
#停止
docker-compose stop
#删除
docker-compose rm
```

  验证节点

```
redis-cli -c -p 6379
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:121.43.162.28
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:27688
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:954733f7a22868fc4a120ca6d12f55c98c59e7a8
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:27688
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:27688
```



## 10.3 redis高可用集群安装

### 10.3.1 安装ruby(有需要)

```
sudo apt-get install ruby

sudo yum install ruby
```

### 10.3.2 创建模板文件

```
mkdir -p /usr/local/docker/redis-cluster
cd /usr/local/docker/redis-cluster && touch redis-cluster.tmpl
```

`redis-cluster.tmpl`内容如下

```

#指定端口
port ${port}
#设置集群可用
cluster-enabled yes
#指定集群生成的配置文件名。注意，这个配置文件不是人为编辑的，是集群在运行中自动生成的，记录着集群中其他节点、状态信息、变量等配置信息，以便在启动的时候能重读到
cluster-config-file nodes.conf
#设置节点最大不可达时间，单位为毫秒。当主节点不可达时间超过这个设置时间，其对应的从节点将替换成为主节点；当一个节点在这个设置时间内不能访问到大多数主节点，将停止接收请求
cluster-node-timeout 5000
#要宣布的IP地址
cluster-announce-ip 0.0.0.0
#要宣布的数据端口
cluster-announce-port ${port}
#要宣布的集群总线端口
cluster-announce-bus-port 1${port}
#设置为aop模式
appendonly yes
#设置槽是否需要全覆盖。默认yes情况下，只要16384个槽没有被全覆盖，整个集群就停止服务。设置no之后，只要还有部分key都继续提供查询处理
cluster-require-full-coverage no
```

三台服务器分别直接敲击命令如下

```
for port in `seq 7001 7002`; do \
  mkdir -p ./${port}/conf \
  && PORT=${port} envsubst < ./redis-cluster.tmpl > ./${port}/conf/redis.conf \
  && mkdir -p ./${port}/data; \
done
```

记得改**cluster-announce-ip 0.0.0.0** 为具体ip

### 10.3.3 docker-compose.yml

```
version: "3.1"
services:
 
  redis-7001:
    image: redis:4.0
    ports: 
      - "7001:7001"
      - "17001:17001"
    volumes: 
      - "/usr/local/docker/redis-cluster/7001/conf/redis.conf:/usr/local/etc/redis/redis.conf"
      - "/usr/local/docker/redis-cluster/7001/data:/data"
    container_name: redis-cluster-7001
    restart: always
    command: redis-server /usr/local/etc/redis/redis.conf
 
  redis-7002:
    image: redis:4.0
    ports: 
      - "7002:7002"
      - "17002:17002"
    volumes: 
      - "/usr/local/docker/redis-cluster/7002/conf/redis.conf:/usr/local/etc/redis/redis.conf"
      - "/usr/local/docker/redis-cluster/7002/data:/data"
    container_name: redis-cluster-7002
    restart: always
    command: redis-server /usr/local/etc/redis/redis.conf
```

```
cd /usr/local/docker/redis-cluster
#启动
docker-compose up -d
#停止
docker-compose stop
#删除
docker-compose rm
```

至此三主三从全部启动完毕，下面需要关联，原始安装命令太过费劲，用官方推荐的`redis-trib`（ruby实现的）

```
redis-cli -c -h 121.43.162.28 -p 7001
redis-cli -c -h 121.43.162.28 -p 7002

redis-cli -c -h 47.96.100.166 -p 7001
redis-cli -c -h 47.96.100.166 -p 7002

redis-cli -c -h 118.24.136.23 -p 7001
redis-cli -c -h 118.24.136.23 -p 7002
```



### 10.3.4  关联三主三从，分配卡槽

```
因为最新的5.0 不再推荐使用ruby，所以拿redis-stable/src/redis-trib.rb操作不了，我又找不到老版本的在线rb文件，所以用下面给的吧
docker run -it --rm ruby sh -c '\
  gem install redis --version=4.0.0\
  && wget http://download.redis.io/redis-stable/src/redis-trib.rb \
  && ruby redis-trib.rb create --replicas 1 \
  121.43.162.28:7001 \
  121.43.162.28:7002 \
  47.96.100.166:7001 \
  47.96.100.166:7002 \
  118.24.136.237:7001 \
  118.24.136.237:7002'
```

```
docker run -it --rm ruby sh -c '\
  gem install redis --version=4.1.0\
  && wget https://github.com/antirez/redis/archive/4.0.11.tar.gz \
  && tar -zxvf 4.0.11.tar.gz\
  && cd redis-4.0.11/src\
  && ruby redis-trib.rb create --replicas 1 \
  121.43.162.28:7001 \
  121.43.162.28:7002 \
  47.96.100.166:7001 \
  47.96.100.166:7002 \
  118.24.136.237:7001 \
  118.24.136.237:7002'
```

```
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
121.43.162.28:7001
47.96.100.166:7001
118.24.136.237:7001
Adding replica 47.96.100.166:7002 to 121.43.162.28:7001
Adding replica 118.24.136.237:7002 to 47.96.100.166:7001
Adding replica 121.43.162.28:7002 to 118.24.136.237:7001
M: 6a8aaa3a2b5bbb5c768db9a2b5be6b653a060507 121.43.162.28:7001
   slots:0-5460 (5461 slots) master
S: 8f5b062d6cfc741f93dc4268bfdb264c922a1d4b 121.43.162.28:7002
   replicates 452d3091dcb62e479e2f823c849df40a7d01f9ed
M: c4885c0bc06e417bed929699147c41aa88a98aa9 47.96.100.166:7001
   slots:5461-10922 (5462 slots) master
S: 95f70de1ed840fbee1ef6d07b255ea974299bf7e 47.96.100.166:7002
   replicates 6a8aaa3a2b5bbb5c768db9a2b5be6b653a060507
M: 452d3091dcb62e479e2f823c849df40a7d01f9ed 118.24.136.237:7001
   slots:10923-16383 (5461 slots) master
S: 6d771e7ac6592b0d4d7c6e7812977cf7713b1ac3 118.24.136.237:7002
   replicates c4885c0bc06e417bed929699147c41aa88a98aa9
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 121.43.162.28:7001)
M: 6a8aaa3a2b5bbb5c768db9a2b5be6b653a060507 121.43.162.28:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: c4885c0bc06e417bed929699147c41aa88a98aa9 47.96.100.166:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 95f70de1ed840fbee1ef6d07b255ea974299bf7e 47.96.100.166:7002
   slots: (0 slots) slave
   replicates 6a8aaa3a2b5bbb5c768db9a2b5be6b653a060507
S: 6d771e7ac6592b0d4d7c6e7812977cf7713b1ac3 118.24.136.237:7002
   slots: (0 slots) slave
   replicates c4885c0bc06e417bed929699147c41aa88a98aa9
M: 452d3091dcb62e479e2f823c849df40a7d01f9ed 118.24.136.237:7001
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 8f5b062d6cfc741f93dc4268bfdb264c922a1d4b 121.43.162.28:7002
   slots: (0 slots) slave
   replicates 452d3091dcb62e479e2f823c849df40a7d01f9ed
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered. 
```



### 10.3.5 集群检查

```
docker run -it --rm ruby sh -c '\
  gem install redis --version=4.1.0\
  && wget https://github.com/antirez/redis/archive/4.0.11.tar.gz \
  && tar -zxvf 4.0.11.tar.gz\
  && cd redis-4.0.11/src\
  && ruby redis-trib.rb check 121.43.162.28:7001'

```

```
>>> Performing Cluster Check (using node 121.43.162.28:7001)
M: 6a8aaa3a2b5bbb5c768db9a2b5be6b653a060507 121.43.162.28:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: c4885c0bc06e417bed929699147c41aa88a98aa9 47.96.100.166:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 95f70de1ed840fbee1ef6d07b255ea974299bf7e 47.96.100.166:7002
   slots: (0 slots) slave
   replicates 6a8aaa3a2b5bbb5c768db9a2b5be6b653a060507
S: 6d771e7ac6592b0d4d7c6e7812977cf7713b1ac3 118.24.136.237:7002
   slots: (0 slots) slave
   replicates c4885c0bc06e417bed929699147c41aa88a98aa9
M: 452d3091dcb62e479e2f823c849df40a7d01f9ed 118.24.136.237:7001
   slots:10923-16383 (5461 slots) master
   0 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```



### 10.3.5 info查看集群信息

```
docker run -it --rm ruby sh -c '\
  gem install redis --version=4.1.0\
  && wget https://github.com/antirez/redis/archive/4.0.11.tar.gz \
  && tar -zxvf 4.0.11.tar.gz\
  && cd redis-4.0.11/src\
  && ruby redis-trib.rb info 121.43.162.28:7001'
```



# 11 、docker compose 安装SpringBoot

##  11.1 创建dockerfile文件

```
FROM openjdk:8-jre
RUN mkdir /app
COPY SpringCloud-05-Config-Server-0.0.1-SNAPSHOT.jar /app/
CMD java -jar /app/SpringCloud-05-Config-Server-0.0.1-SNAPSHOT.jar  --spring.profiles.active=peer0
EXPOSE 8888
```

## 11.2 创建docker-compose.yml文件

```
version: "3.1"

services:

  spring-cloud-config-8888: 
    build:
      context: ./
      dockerfile: Dockerfile-config-8888
    restart: always     
    container_name: spring-cloud-config-8888
    ports:
      - "8888:8888"    
```

## 11.3 项目发布

### 11.3.1 编译

```
docker-compose build
```

### 11.3.2 启动

```
docker-compose up -d
```

### 11.3.3 停止

```
docker-compose stop
docker-compose rm
```

## 12、安装kafka

### 12.1、 单机



### 12.2、真实集群

> kafka 0.9.x以后的版本不要使用 advertised.host.name 和 advertised.host.port 已经deprecated

#### 第一台主机

```
version: '3.1'

services:

  kafka:
    image: wurstmeister/kafka:2.12-2.1.0
    restart: always
    container_name: kafka
    ports:
      - "9092:9092"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 121.43.162.28:2181,47.96.100.166:2181,118.24.136.237:2181
      KAFKA_LISTENERS: PLAINTEXT://:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://121.43.162.28:9092


  kafka-manager:
    image: sheepkiller/kafka-manager:latest
    container_name: kafka-manager
    ports:
      - "9000:9000"
    environment:
      ZK_HOSTS: 121.43.162.28:2181,47.96.100.166:2181,118.24.136.237:2181
      APPLICATION_SECRET: letmein
      KM_ARGS: -Djava.net.preferIPv4Stack=true
```

#### 第二台主机

```
version: '3.1'

services:

  kafka:
    image: wurstmeister/kafka:2.12-2.1.0
    restart: always
    container_name: kafka
    ports:
      - "9092:9092"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: 121.43.162.28:2181,47.96.100.166:2181,118.24.136.237:2181
      KAFKA_LISTENERS: PLAINTEXT://:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://47.96.100.166:9092


  kafka-manager:
    image: sheepkiller/kafka-manager:latest
    container_name: kafka-manager
    ports:
      - "9000:9000"
    environment:
      ZK_HOSTS: 121.43.162.28:2181,47.96.100.166:2181,118.24.136.237:2181
      APPLICATION_SECRET: letmein
      KM_ARGS: -Djava.net.preferIPv4Stack=true
```

#### 第三台主机

```
version: '3.1'

services:

  kafka:
    image: wurstmeister/kafka:2.12-2.1.0
    restart: always
    container_name: kafka
    ports:
      - "9092:9092"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: 121.43.162.28:2181,47.96.100.166:2181,118.24.136.237:2181
      KAFKA_LISTENERS: PLAINTEXT://:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://118.24.136.237:9092


  kafka-manager:
    image: sheepkiller/kafka-manager:latest
    container_name: kafka-manager
    ports:
      - "9000:9000"
    environment:
      ZK_HOSTS: 121.43.162.28:2181,47.96.100.166:2181,118.24.136.237:2181
      APPLICATION_SECRET: letmein
      KM_ARGS: -Djava.net.preferIPv4Stack=true

```

查看kafka版本

```
find / -name \*kafka_\* | head -1 | grep -o '\kafka[^\n]*'
```

