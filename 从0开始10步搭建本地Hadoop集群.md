## 前言

相信每一位大数据时代的程序员朋友，对Hadoop这个将 “分而治之”思想体现得淋漓尽致的**分布式处理框架**并不陌生（~~指听过~~） 。

作为一名入行不久的Data Engineer ，本着看一看，学一学，玩一玩的三步走战略，利用闲暇时间几经周折终于把这位大名鼎鼎的Hadoop，请到了我的电脑上。

对于Hadoop更多的基础概念，组件构成，应用场景等等理论知识，已经有 Miao Pengfei同学的文章 “#博客大赛# MacOS 本地安装 hadoop 集群填坑之路” 珠玉在前，本文不再赘述。

所以本文将重点记录从创建虚拟网络开始的Hadoop的搭建过程， 力求事无巨细，开箱即用。

## 环境

- 处理器：2.6GHz 6-Core Intl Core i7
- Docker version: 20.10.16
- Colima version:  0.4.2
- Docker config:
    - Cpu 4核
    - Memory 10G
    - Disk 10G

## 正文


### 1. 为Hadoop创建虚拟网络
```sh 
docker network create --driver=bridge hadoop
```
- 查看创建的网络
```sh
docker network ls
```

![](https://fastly.jsdelivr.net/gh/filess/img5@main/2022/11/27/1669526024949-612cc21f-63f3-4740-bb06-abaa831a049c.png)

### 2. 拉取 `ubuntu`镜像
```sh
docker pull ubuntu:16.04
```

### 3. 进入 `ubuntu`镜像
```sh
docker run -it ubuntu:16.04 /bin/bash
```

### 4. 更新源文件
- 备份旧源
```sh
cp /etc/apt/sources.list /etc/apt/sources_init.list
```
- 删除旧源
```sh
rm /etc/apt/sources.list
```

- 添加新源至`/etc/apt/sources.list`  (利用`echo "" >> /etc/apt/sources.list`)

```sh
deb http://mirrors.aliyun.com/ubuntu/ xenial main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main

deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main

deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates universe

deb http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security universe
```

- 更新源
```sh
apt update
```
### 5. 安装Java及基础工具

 ```sh
 apt install openjdk-8-jdk
 ```
安装完成之后，使用：`java -version` 来查看是否安装成功，显示 java版本号就是成功了。

- 安装`scala`（方便玩耍spark，建议一起装了）/ `vim`/`net-tools`/`openssh-server`/`openssh-client`：
  ```sh
 apt install scala
 apt install vim
 apt install net-tools
 apt install openssh-server
 apt install openssh-client
 ```
### 6. 配置免密登陆

- 进入根目录
```sh
cd ~
```
- 生成密钥 (不考虑安全问题的话一路回车即可。密钥存于当前用户根目录下的 `.ssh` 文件夹中）
```sh
ssh-keygen -t rsa -P ""
```
- 将公钥注入 `authorized_keys` 文件中
```sh
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
```
- 启动 SSH 服务
```sh
service ssh start
```
- 免密登录自己
```sh
ssh 127.0.0.1
```

可看到日志：
```sh
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-45-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Tue Mar 19 07:46:14 2019 from 127.0.0.1
root@fab4da838c2f:~#
```

- 修改 `.bashrc` 文件，配置当启动 shell 的时候，自动启动 SSH 服务。 --> 用 vim 打开 `.bashrc` 文件，在文末添加`service ssh start`  (记得sourece 一下以激活修改)

#### 至此，虚拟机初始化完毕

p.s.  这篇博客一直被Gmail限制，死活都发不出来，折腾半天无果，所以我把他一分为二，下半篇以回复的形式补充进来，希望大家理解TAT

--- 

### 7. 下载并安装hadoop源文件
- 下载
```sh
wget http://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz
```
- 切换目录并重命名
```sh
cd ~
tar -zxvf hadoop-3.3.2.tar.gz -C /usr/local/
cd /usr/local/
mv hadoop-3.3.2 hadoop       
```

### 8. 添加环境变量

编辑环境变量文件：
```sh
vim /etc/profile
```
并在该文件末尾添加如下内容：
```sh
#java
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
#hadoop
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_LIBEXEC_DIR=$HADOOP_HOME/libexec
export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native:$JAVA_LIBRARY_PATH
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HDFS_DATANODE_USER=root
export HDFS_DATANODE_SECURE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export HDFS_NAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```



### 9. 配置Hadoop环境
#### 重点！ 重点！！ 重点！！！
Hadoop中有6个文件是需要重点配置的，分别是： `hadoop-env.sh`, `core-site.xml`， `hdfs-site.xml`， `mapred-site.xml`，  `yarn-site.xml`，  `workers`
下面就一个个开始配置

- 修改 `hadoop-env.sh` 文件，在文件末尾添加以下信息：
```sh
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

- 修改 `core-site.xml` 为：
```shell
<configuration>　　<!--指定nameNode的地址-->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://h01:8020</value>
    </property>
　　<!--指定Hadoop数据的存储目录-->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/data</value>
    </property>
　　<!--配置HDFS网页登陆使用的静态用户，配置这个之后才有权限可以在网页端删除文件、文件夹-->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>root</value>
    </property>
</configuration>
```

- 修改 `hdfs-site.xml` 为：
```shell
<configuration>　　<!--文件的存储个数-->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>　　<!--nn web端访问地址，使用网页访问HDFS文件系统就是这个端口-->
    <property>
        <name>dfs.namenode.http-address</name>
        <value>h01:9870</value>
    </property>　　<!--2nn web端访问地址-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>h01:9868</value>
    </property>　　<!--网页查看HDFS文件内容，出现Couldn‘t preview the file报错，需要配置的参数-->
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
</configuration>
```

- 修改 `mapred-site.xml` 为：
```shell
<configuration>　　<!--指定MapReduce程序运行在Yarn上-->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

- 修改 `yarn-site.xml` 为：
```shell
<configuration>
　　<!--指定MR走 shuffle-->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
　　<!--指定ResourceManager的地址-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>h01</value>
    </property>
　　<!--环境变量的继承-->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>         
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

- 修改`workers`为:
```sh
h01
h02
h03
```
计划起几个集群就写到几，注意**行末不要有空格**，**最后不要有空行**

### 10. 在Docker中启动集群

- 先将当前容器导出为镜像(注意使用自己的ubuntu容器id 可通过docker-ps 查看)

![](https://fastly.jsdelivr.net/gh/filess/img8@main/2022/11/27/1669528960674-42e76f2c-2e50-42a1-aa92-22509af74bbf.png)

```sh
docker commit 66b072cc4323 newhadoop
```

可通过`docker images`查看更新后的镜像文件
![](https://fastly.jsdelivr.net/gh/filess/img1@main/2022/11/27/1669529002606-ed7a2c6d-9a82-40bb-a8b1-708ad4e89198.png)

- 再**分别**启动三个终端
```sh
docker run -it --network hadoop -h h01 --name "h01" -p 9870:9870 -p 8088:8088 newuhadoop /bin/bash
```
第一条命令启动的是` h01 `是做 `master` 节点的，所以暴露了端口，以供访问 web 页面。

后面两条无需额外设置

```sh
docker run -it --network hadoop -h h02 --name "h02" newuhadoop /bin/bash
```

```sh
docker run -it --network hadoop -h h03 --name "h03" newuhadoop /bin/bash
```

## 一切准备完毕，开冲！

### 在 `h01` 主机中，启动 Haddop 集群
- 先进行格式化操作（**只有第一次启动**的时候需要初始化, 不然hdfs起不来）：
  `root@h01:/usr/local/hadoop# ./bin/hdfs namenode -format`
- 然后启动HDFS集群：
  `root@h01:/usr/local/hadoop# ./sbin/start-dfs.sh`
- 最后，启动yarn集群管理节点：
  `root@h01:/usr/local/hadoop# ./sbin/start-yarn.sh`

-  启动完成，使用`jps` 命令查看：
```shell
root@h01:/usr/local/hadoop# jps
36435 Jps
36087 NodeManager
35959 ResourceManager
35306 SecondaryNameNode
34970 NameNode
35100 DataNode
root@h01:/usr/local/hadoop#
```

浏览器访问本机的9870端口，可以看到Hadoop的文件管理系统：

![](https://fastly.jsdelivr.net/gh/filess/img1@main/2022/11/27/1669530052645-bb1de50d-4caa-4f4a-9cee-a7093d78ecfa.png)

浏览器访问本机的8088端口，可以看到Hadoop中 Yarn的资源调度系统：

![](https://fastly.jsdelivr.net/gh/filess/img9@main/2022/11/27/1669530349400-4518b834-ca44-46a4-bfac-d407311b1e9d.png)
