# 0 本机环境

M1 Pro

虚拟机：VMware Fusion

CentOS Stream 9

# 1 虚拟机设置

## 1.1 下载好 Stream 9，创建新的虚拟机

下载地址：[Index of /9-stream (centos.org)](https://mirror.stream.centos.org/9-stream/)





<img src="https://github.com/pcpy007/pcpy007.github.io/blob/master/img/image-20230705185144380.png" alt="image-20230705185144380" style="zoom:50%;" />

## 1.2 操作系统对应 RHEL 9

<img src="https://github.com/pcpy007/pcpy007.github.io/blob/master/img/image-20230705185035765.png" alt="image-20230705185035765" style="zoom:50%;" />



## 1.3 网络适配器选择桥接模式

![image-20230705184803664](https://github.com/pcpy007/pcpy007.github.io/blob/master/img//image-20230705184803664.png)

## 1.4 设置 IP，关闭防火墙

```shell
#配置 ip
cd /etc/NetworkManager/system-connections/
ls
vi ens160.nmconnection

#查看防火墙状态
systemctl status firewalld
#关闭防火墙
systemctl stop firewalld
systemctl disable firewalld.service
```

再新增两个虚拟机，完成如上配置

## 1.5 安装软件

通过 FTP 工具上传软件，Mac 端无 Xshell 及 Xftp，可以使用 Royal TSX

上传 JDK, Hadoop 相关压缩包

```shell
tar -zxvf jdk-8u361-linux-aarch64.tar.gz
tar -zxvf hadoop-3.3.5.tar.gz
```

## 1.6 设置环境变量

```shell
vi ~/profile
export JAVA_HOME=/opt/module/jdk1.8.0_361
export PATH=$PATH:$JAVA_HOME/bin
export HADOOP_HOME=/opt/module/hadoop-3.3.5
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

## 1.7 修改 hosts

```shell
sudo vi /etc/hosts

192.168.3.101	hadoop101
192.168.3.102	hadoop102
192.168.3.103	hadoop103
```

## 1.8 配置 ssh 免密登录

在每台机器上执行

```shell
ssh-keygen -t rsa

ssh-copy-id hadoop101
ssh-copy-id hadoop102
ssh-copy-id hadoop103
```



## 1.9 编写集群同步工具脚本

基于 rsync

```shell
cd /home/nannan7
mkdir bin
vi xsync

#写入下面的脚本
#!/bin/bash

#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi
#2. 遍历集群所有机器
for host in hadoop101 hadoop102 hadoop103
do
    echo ====================  $host  ====================
    #3. 遍历所有目录，挨个发送

    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e $file ]
            then
                #5. 获取父目录
                pdir=$(cd -P $(dirname $file); pwd)

                #6. 获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done
```

后续，基于以上工具，在各台机器间同步文件

```shell
xsync /opt/module/hadoop-3.3.5

#单个文件使用下列命令同步
scp -r ./profile root@hadoop102:/root/
```

# 2 配置 Hadoop

三台机器组成的集群，其分工如下：

![image-20230705213619124](/Users/pcpy007/Library/Application Support/typora-user-images/image-20230705213619124.png)

## 2.1 配置 hadoop-env.sh

路径: hadoop 安装目录，./etc/hadoop/hadoop-env.sh

```shell
export JAVA_HOME=
export HADOOP_HOME=
```

## 2.2 配置 core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop101:9000</value>
    </property>
</configuration>
```

## 2.3 配置 hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop101:9870</value>
    </property>
   <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop103:9868</value>
    </property>
</configuration>
```

## 2.4 配置 yarn-site.xml

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop102</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>  <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

## 2.5 配置 mapred-site.xml

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name> <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

## 2.6 配置 workers

路径: hadoop 安装目录，etc/hadoop/workers

```shell
vi etc/hadoop/workers

#加入下面几行
hadoop101
hadoop102
hadoop103
```

将以上配置，使用 xsync 同步至另外两台机器上

# 3 启动集群

## 3.1 执行命令

```shell
#hadoop101
cd /opt/module/hadoop-3.3.5
sbin/start-dfs.sh

#hadoop102
cd /opt/module/hadoop-3.3.5
sbin/start-yarn.sh
```

## 3.2 访问 web 页面

hdfs 页面：[hadoop101](http://hadoop101:9870/)

<img src="https://github.com/pcpy007/pcpy007.github.io/blob/master/img/image-20230705214600953.png" alt="image-20230705214600953" style="zoom:50%;" />

yarn 后台页面：http://hadoop102:8088/

<img src="https://github.com/pcpy007/pcpy007.github.io/blob/master/img/image-20230705214846694.png" alt="image-20230705214846694" style="zoom:50%;" />

至此，初步可见集群搭建完成。