# Hadoop环境搭建

## 1 虚拟机准备

### 1.1 待克隆虚拟机准备

#### 1.1.1 yum 更新

```shell
yum update -y
```

2. 安装java环境

   1. 通过SFTP上传JDK,如果是rpm包则通过以下命令安装。安装完路径为 

      ```
      rpm -ivh jdk-8u91-linux-x64.rpm
      ```

   2. 配置环境变量

      ```
      # vim 大写G则跳转至最后一行
      export JAVA_HOME=/usr/java/jdk1.8.0_91
      export PATH=$PATH:$JAVA_HOME/bin
      ```

### 1.2 克隆Centos7

1. VMware打开库
2. 虚拟机右键->管理->克隆->创建完整克隆

### 1.3 修改克隆虚拟机的静态IP

1. 安装的过程需要注意开启网络

2. 修改静态配置文件

   ```
   vi /etc/sysconfig/network-scripts/ifcfg-ens33
   IPADDR=
   NETMASK=
   GATEWAY=
   # DNS没有配对可能访问不了互联网
   DNS1=
   DNS2=
   ```

### 1.4 配置hosts文件

```
# 注意静态地址不要写错
192.168.5.100 hadoop1
192.168.5.101 hadoop2
192.168.5.102 hadoop3

# 重启网络
systemctl restart network
```

### 1.5 关闭防火墙

### 1.6 



## 2. SSH连接工具

### 2.1  SecureCRT

多容器命令同时输入

window->Tile Horizontally

底部右键->command Window->底部右键->send command to all sessions

### 2.2 clustershell

1. 集群免密访问

   ```
   1. 生成ssh
   ssh-keygen -t rsa
   2. 查看目录 .ssh
   ls /rooot/.ssh
   # 存在 id_rsa id_rsa.pub两个文件
   3. 集群拷贝
   ssh-copy-id -i /root/.ssh/id_rsa.pub 
   4. 查看是否添加成功
   more /root/.ssh/authorized_keys
   5. 验证
   ssh hadoop1
   6. 在当前环境下查看是否所有集群都在hadoop1
   hostname
   ```

2. 安装clustershell

   ```shell
   yum --enablerepo=extras install epel-release
   yum install clustershell -y
   
   # 在/etc/clustershell/groups文件
   hadoop: hadoop[1-3]
   ```

3. 使用clustershell

```
clush -g hadoop -c /opt/module/hadoop-2.9.2/
```



## 3. Hadoop集群安装

### 3.1 解压并配置Hadoop环境变量

```
1. 添加环境变量配置
export HADOOP_HOME=/opt/module/hadoop-2.9.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

2.刷新
source /etc/profile
```



### 3.2 了解Hadoop文件目录

![](C:\Users\Lch Mag D\Desktop\大数据\img\Hadoop文件目录.png)

1. bin目录

   ![](C:\Users\Lch Mag D\Desktop\大数据\img\Hadoop bin.png)

2. sbin目录

   Hadoop集群停止、启动命令

   ![](C:\Users\Lch Mag D\Desktop\大数据\img\sbin.png)

3. 配置文件在 hadoop-2.9.2/etc/hadoop

   ![](C:\Users\Lch Mag D\Desktop\大数据\img\配置文件.png)

4. share目录

   存放一些测试的jar包

### 3.3 Hadoop单机节点启动 测试

1. 创建 input目录

2. 执行命令

   ```
   hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar grep input/ output 'dfs[a-z.]+'
   ```

3. 统计个数

   ```
   hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wcount input/ output/
   ```

   ![](C:\Users\Lch Mag D\Desktop\大数据\img\执行成功文件.png)

![](C:\Users\Lch Mag D\Desktop\大数据\img\词汇统计.png)



### 3.4 HDFS

#### 3.4.1 DateNode和NameNode进程同时只能有一个工作问题

1. NameNode 在格式化初始化后会生成clusterId(集群Id),DateNode在启动后也会生成与NameNode一样的Id,如果NameNode再次格式化后,新生成的clusterId与原有的不匹配,
2. 解决方式: 在格式化之前, 先删除DataNode里面的信息,默认在/tmp,如果配置了该目录, 则去配置的目录删除数据
3. VERSION目录为 `/opt/module/hadoop-2.9.2/data/tmp/dfs/name/current`

### 3.5 Yarn

#### 3.5.1 配置yarn

1. 配置yarn-env.sh

```
export JAVA_HOME=
```

2. 配置yarn-site.xml

```xml
<configuration>
    <!-- 设置 resourcemanager 在哪个节点-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>namenode</value>
    </property>

	<!--Reducer获取数据的方式-->	
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

3. 配置mapred-site.xml

```xml
<configuration>
    <property>
    	<!--指定MR运行在YARN上,默认有local-->
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

3. 单节点启动

   ```
   yarn-daemon.sh start resourcemanager
   yarn-daemon.sh start nodemanager
   ```

   

### 3.6 配置文件说明

1. 默认配置文件

   ![](C:\Users\Lch Mag D\Desktop\大数据\img\默认.png)



### 3.7 集群配置

#### 3.7.1 集群部署规划

|        | hadoop1                                              | hadoop2                                              | hadoop3                                              |
| ------ | ---------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| HDFS   | NameNode(NN) (9000) DataNode(DN)                     | DataNode                                             | SecondaryNameNode(50090) DataNode                    |
| YARN   | NodeManager                                          | ResourceManager (8088) NodeManager                   | NodeManager                                          |
| 实例图 | ![](C:\Users\Lch Mag D\Desktop\大数据\img\实例1.png) | ![](C:\Users\Lch Mag D\Desktop\大数据\img\实例2.png) | ![](C:\Users\Lch Mag D\Desktop\大数据\img\实例3.png) |

![](C:\Users\Lch Mag D\Desktop\大数据\img\webinterface.png)

#### 3.7.2 [配置集群](https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-common/ClusterSetup.html)

1. 配置`hadoop-env.sh`

   ``` bash
   export JAVA_HOME=/usr/java/jdk1.8.0_91
   export HADOOP_PID_DIR=/opt/module/hadoop-2.9.2/tmp
   # 其实还可以设置更多的东西,比如配置环境变量,如节点使用的垃圾收集器 等
   export HADOOP_NAMENODE_OPTS="-XX:+UseParallelGC"
   ```

2. 配置`core-site.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
		<!-- 指定HDFS中NameNode的地址 hdfs://host:port-->
		<name>fs.defaultFS</name>
		<value>hdfs://hadoop1:9000</value>
	</property>
	<property>
		<!-- 指定Hadoop运行时产生文件的存储目录 -->
		<name>hadoop.tmp.dir</name>
		<value>/opt/module/hadoop-2.9.2/data/tmp</value>
	</property>
	<property>
		<!-- 序列化文件 -->
		<name>io.file.buffer.size</name>
		<value>131072</value>
	</property>
</configuration>
```

3. 配置`hdfs-site.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
       <!--配置HDFS second NAMENODE 节点 --> 
		<name>dfs.namenode.secondary.http-address</name>
		<value>hadoop3:50090</value>
	</property>
</configuration>
```

4. 修改yarn-env.sh,添加JAVA_HOME
5. 配置`yarn-site.xml`

```xml
<?xml version="1.0"?>
<configuration>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<property>
		<name>yarn.resourcemanager.address</name>
		<value>hadoop2</value>
	</property>
</configuration>
```

6. 修改`mapred-env.sh`
7. 配置`mapred-site.xml`
8. 在`/etc/hadoop/slaves`下添加

```
hadoop1
hadoop2
hadoop3
```

### 3.8 启动集群

#### 3.8.1 格式化HDFS

```shell
# 格式化DFS文件系统
/bin/hdfs namenode -format
```

#### 3.8.2 启动HDFS

```shell
sbin/start-dfs.sh
```

#### 3.8.3 在ResourceManager节点上启动`YARN`

```
sbin/start-yarn.sh
```







































