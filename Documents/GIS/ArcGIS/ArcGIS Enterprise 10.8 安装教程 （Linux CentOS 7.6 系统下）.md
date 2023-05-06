## 1. 基础环境信息设置

### 1.1 操作系统

Linux 操作系统版本：CentOS 7.6

软件：arcgis 10.8 portal\server\datastore(分别部署在三台服务器上）

192.168.0.xx6 portal.esriwhu.cn

192.168.0.xx7 server.esriwhu.cn

192.168.0.xx8 data.esriwhu.cn


![ArcGIS Enterprise 10.8 安装教程 （Linux CentOS 7.6 系统下）-1.png - Droplr](https://d.pr/i/GDsCS3+)

### 1.2 创建 arcgis 账户

```shell
[root@portal home]#  groupadd arcgis

[root@portal home]#  useradd -g arcgis -m arcgis

[root@portal home]#  passwd arcgis

```

### 1.3 修改server机器名
```shell
[root@portal ~]# vi /etc/hostname
```

重启

```shell
[root@server ~]# hostname
```

### 1.4 修改 host 文件

每个节点都要加入整个集群及其的 hosts 信息

```
[root@portal ~]# vi /etc/hosts
```

关闭防火墙

```shell
[root@server ~]# sudo systemctl stop firewalld
[root@server ~]# systemctl status firewalld
```

![ArcGIS Enterprise 10.8 安装教程 （Linux CentOS 7.6 系统下）-2.png - Droplr](https://d.pr/i/U43cLj+)

如果是想重启后防火墙还是处于关闭的状态，得使用命令：
```shell
sudo systemctl disable firewalld
```


### 1.5 禁用ipv6

```shell
[root@portal ~]# vi /etc/sysctl.conf

net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

[root@portal ~]#sysctl -p
```

### 1.6 修改文件句柄数

```shell
[root@portal ~]# vi /etc/security/limits.conf

arcgis soft nofile 65535
arcgis hard nofile 65535
arcgis soft nproc 25059
arcgis hard nproc 25059
```

关系和切片缓存数据存储的最小文件句柄限制为65,535，时空大数据存储的最小文件句柄限制为65,536。所有数据存储类型的最小处理限制为25,059。这些最低设置仅可确保ArcGIS Data Store可以启动。您应该设置更高的限制，以帮助确保系统继续运行。

要增加软限制和硬限制，请使用超级用户访问权限来编辑/etc/security/limits.conf文件。有关时空大数据存储计算机的/etc/security/limits.conf文件设置

### 1.7 在 Spatiotemporal DataStore 各节点上修改 swap 值

在配置/etc/sysctl.conf添加内容：

```shell
vm.max_map_count=262144
vm.swappiness=1
```

### 1.8 JDK 环境变量配置

华为镜像：[https://repo.huaweicloud.com/java/jdk/8u201-b09/](https://repo.huaweicloud.com/java/jdk/8u201-b09/)
Root账户下
```shell
tar -zxvf dk-8u201-linux-x64.tar.gz
```

编辑环境变量配置文件 vi /etc/profile，添加如下内容：

```shell
export JAVA_HOME= /home/arcgis/jdk1.8.0_201
export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH

source /etc/profile

``` 

利用命令查看是否生效：

```shell
[root@server arcgis]# source /etc/profile
[root@server arcgis]# javac
```


### 1.9 安装tomcat

```shell
tar -zxvf  apache-tomcat-9.0.21.tar.gz
```


### 1.10 Tomcat 配置 https

首先需要进行 tomcat 开启 https 配置，这里使用自签名证书。

利用Java的keytool命令，按照提示进行参数输入：
```shell
keytool -genkey -alias tomcat -keyalg RSA -validity 36500 -keystore /home/arcgis/tomcat9 /tomcat.keystore -keysize 2048
```

## 2 Web Adaptor 安装

切换到 arcgis 用户，导航到软件安装目录，执行安装：
其中参数 –m，是选择安装模式，-l，是对安装协议是否同意：
```shell
Setup -m silent -l yes  -c /home
```


安装完成以后，等待注册 Server、和 Portal。

WebAdaptor 安装完成以后，将 WebAdaptor 安装目录下的 arcgis.war 包复制到tomcat 的 webapps 中。

重启 tomcat 服务器，在浏览其中访问看是否能够访问：（图示能够访问，说明webadaptor 安装成功）

## 3. Portal for ArcGIS 安装

### 3.1 解压安装

1.  解压软件：
```shell
tar -zxvf /home/arcgis/Portal_for_ArcGIS_Linux_108_172356.tar.gz -C /home/arcgis
```
3.  解压完成以后，切换到 arcgis 账户，进入到安装目录
4.  检验环境是否符合安装 portal
```shell
[arcgis@portal root]$ cd /home/arcgis/PortalForArcGIS/portaldiag
```

### 3.2授权并创建账户

### 3.3 配置portal&Web Adaptor

以 arcgis 用户导航到/home/arcgis/webadaptor10.8/java/tools安装目录，执行命进行 Portal 注册：
```shell
./configurewebadaptor.sh -m portal -w [https://xxxxxxxx/arcgis/webadaptor](https://xxxxxxxx/arcgis/webadaptor) -g [https://SSSSSSS:7443](https://SSSSSSS:7443) -u arcgis -p esri1234
```


其中参数含义：
-m： 注册的是 server，还是 portal
-w：输入 WebAdaptor 地址
-g：输入访问地址
-u,-p：输入用户名和密码

## 4、ArcGIS Server 安装配置

### 4.1 安装前验证（注意：需要根据报错修改）

```shell
cd /home/arcgis/ArcGISServer/serverdiag
[arcgis@server serverdiag]$ ./serverdiag
```

 安装授权（注意：安装和授权可以同时进行）
 ```shell
Setup -m silent -l yes -a /path/to/server.ecp #（许可文件路径） 
```
 

注释：具体解释
![ArcGIS Enterprise 10.8 安装教程 （Linux CentOS 7.6 系统下）-4.png - Droplr](https://d.pr/i/J2BmJj+)

```shell
Setup --mode silent --license-agreement yes --authorization-file /path/to/server.ecp

This will run the installer in Silent mode, using the installation directory /opt/arcgis/server.

 Setup -m silent -l yes -a /path/to/server.ecp -d /opt/arcgis/server #（许可路径+安装路径）
```


### 4.2 授权帮助说明
![ArcGIS Enterprise 10.8 安装教程 （Linux CentOS 7.6 系统下）-5.png - Droplr](https://d.pr/i/qscHvD+)

### 4.3 创建站点

创建完成登录验证是否可以 [https://server.esriwhu.cn:6443/arcgis/manager](https://server.esriwhu.cn:6443/arcgis/manager)

4.4 配置server&Web Adaptor

在GIS Server执行安装完成以后，在WebAdaptor目录下，通过命令执行注册，以 arcgis 用户导航到/home/arcgis/webadaptor10.8/java/tools(WebAdaptor安装目录)

其中参数含义：

**-m**：注册的是 server，还是 portal；
**-w**：输入 WebAdaptor 地址；
**-g**：输入 Server 访问地址；
**-u,-p**：输入 Server Admin 用户名和密码；
**-a**：设置是否配置 WebAdaptor 管理员权限访问 Server Manager

配置命令：

./configurewebadaptor.sh -m server -w [https://server.esriwhu.cn/arcgis/webadaptor](https://server.esriwhu.cn/arcgis/webadaptor) -g [https://server.esriwhu.cn:6443](https://server.esriwhu.cn:6443) -u siteadmin -p admin1234 -a true

## 5、ArcGIS DataStore 安装

### 5.1 开始安装 datastore，执行命令：

```shell
./Setup -m silent -l yes -d /home
./Setup --mode silent --license-agreement yes
./Setup -m silent -l yes -d /opt/arcgis/datastore
```


### 5.2 Hosted Server 注册 DataStore

导航到DataStore安装目录/tools（/home/arcgis/datastore/tools）输入命令进行数据库注册：
```shell
./configuredatastore.sh [https://server.esriwhu.cn:6443/arcgis/admin](https://server.esriwhu.cn:6443/arcgis/admin) siteadmin admin1234  /home/arcgis/database --stores relational,spatiotemporal,tileCache
```


## 6、Portal 联合 hosting server

### 6.1 联合 hosting server

输入 hosting server 的路径,Services URL 为不带端口的路径，形如：

[https://server.esriwhu.cn/arcgis/manager](https://server.esriwhu.cn/arcgis/manager)

带端口

[https://server.esriwhu.cn:6443/arcgis/manager](https://server.esriwhu.cn:6443/arcgis/manager)

通过浏览器访问 Portal（webadaptor 注册后的地址），
并在编辑设置中添加 Server，输入 Server 地址及用户名和密码：
![ArcGIS Enterprise 10.8 安装教程 （Linux CentOS 7.6 系统下）-6.png - Droplr](https://d.pr/i/TQvEak+)