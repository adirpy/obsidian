
## 取得文件行数

```bash
awk '{print NR}' a|tail -n1
awk 'END{print NR}' a
grep -n "" a|awk -F: '{print '}|tail -n1
sed -n '$=' a
wc -l a|awk '{print }'
cat a |wc -l
```



## 查看Linux内核版本命令（两种方法）

**cat /proc/version**

```bash
[root@localhost ~]# cat /proc/version
Linuxversion 2.6.18-194.8.1.el5.centos.plus (mockbuild@builder17.centos.org) (gcc version 4.1.2 20080704 (Red Hat 4.1.2-48)) #1 SMP Wed Jul 7 11:50:45 EDT 2010
```



**uname -a**

```bash
[root@localhost ~]# uname -a
Linux localhost.localdomain 2.6.18-194.8.1.el5.centos.plus #1 SMP Wed Jul 7 11:50:45 EDT 2010 i686 i686 i386 GNU/Linux
```



## 查看Linux系统版本的命令（3种方法）

**lsb_release -a，即可列出所有版本信息：**

```bash
[root@localhost ~]# lsb_release -a
LSB Version: :core-3.1-ia32:core-3.1-noarch:graphics-3.1-ia32:graphics-3.1-noarch
Distributor ID: CentOS
Description: CentOS release 5.5 (Final)
Release: 5.5
Codename: Final
```



**这个命令适用于所有的Linux发行版，包括Redhat、SuSE、Debian…等发行版。**



**cat /etc/redhat-release，这种方法只适合Redhat系的Linux：**

```bash
[root@localhost ~]# cat /etc/redhat-release
CentOS release 5.5 (Final)
```



**cat /etc/issue，此命令也适用于所有的Linux发行版**

```bash
[root@localhost ~]# cat /etc/issue**
CentOS release 5.5 (Final)
Kernel \r on an \m
Webarchive to html
textutil -convert html xxx.webarchive
```
