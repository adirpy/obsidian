
不容易，终于安装成功了，这里尝试把遇到的所有问题都记录下来，以便日后来查看。主要参考的是以下2个链接:
[http://my.oschina.net/farces/blog/279434?fromerr=9orjH3Zz](http://my.oschina.net/farces/blog/279434?fromerr=9orjH3Zz)
[http://www.linuxidc.com/Linux/2013-02/79079p2.htm](http://www.linuxidc.com/Linux/2013-02/79079p2.htm)

## 1. 将系统更新到最新：

```
sudo apt-get update
sudo apt-get dist-upgrade ##这个可以不要
```

## 2. 安装依赖包

```
sudo apt-get install automake autotools-dev binutils bzip2 elfutils expat gawk gcc gcc-multilib g++-multilib ia32-libs ksh less lesstif2 lesstif2-dev lib32z1 libaio1 libaio-dev libc6-dev libc6-dev-i386 libc6-i386 libelf-dev libltdl-dev libmotif4 libodbcinstq4-1 libodbcinstq4-1:i386 libpth-dev libpthread-stubs0 libpthread-stubs0-dev libstdc++5 lsb-cxx make openssh-server pdksh rlwrap rpm sysstat unixodbc unixodbc-dev unzip x11-utils zlibc
```

这是网上列出的全部的包，但实际中，还是存在不能通过apt-get安装的包，但不能安装的中，大部分都通过以下链接找到了可以安装的包：

[https://launchpad.net/ubuntu](https://launchpad.net/ubuntu)

最后还剩下2个没有安装，但是好像没有啥影响：

```
ia32-libs
libpthread-stubs0
```

## 3. 创建用户

以我自己的习惯为准

```
sudo groupadd oinstall
sudo groupadd dba
sudo useradd -g oinstall -G dba -d /home/oracle -s /bin/bash oracle
sudo passwd oracle
```

## 4. 设置系统变量

```
/sbin/sysctl -a | grep sem
/sbin/sysctl -a | grep shm
/sbin/sysctl -a | grep file-max
/sbin/sysctl -a | grep aio-max
/sbin/sysctl -a | grep ip_local_port_range
/sbin/sysctl -a | grep rmem_default
/sbin/sysctl -a | grep rmem_max
/sbin/sysctl -a | grep wmem_default
/sbin/sysctl -a | grep wmem_max
```

然后根据上面命令中得到的参数值在/etc/sysctl.conf中增加对应数据，比如：

```
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 536870912
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
```

运行一下命令更新内核参数：

```
sysctl –p
```

添加对oracle用户的内核限制，在/etc/security/limits.conf文件中增加以下数据:

```
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024oracle hard nofile 65536
oracle soft stack 10240
```

查看/etc/pam.d/login，增加以下行（有了就不用增加了）：

```
session required pam_limits.so
```

同样检查/etc/pam.d/su，没有以下行就自己加上：

```
session required pam_limits.so
```

## 5. 创建oracle文件夹

```
mkdir -p /home/oracle
mkdir -p /home/oracle/oraInventory
chown -R oracle:oinstall /home/oracle
chown -R oracle:oinstall /home/oracle/oraInventory
```

## 6. 配置oracle环境变量

为Oracle配置环境变量。同样在主文件夹下的.bashrc配置文件中加入如下内容：

```
export ORACLE_BASE=/opt/oracle 
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1
export ORACLE_SID=orcl
export ORACLE_UNQNAME=orcl
export NLS_LANG=.AL32UTF8
export PATH=${PATH}:${ORACLE_HOME}/bin/
```

## 7. 配置链接

oracle本身并不支持ubuntu来安装，所以要进行欺骗oracle的安装程序（sudo执行）：

* 针对32位系统：

```
ln -s /etc /etc/rc.d
ln -s /lib/i386-linux-gnu/libgcc_s.so.1 /lib/
ln -s /usr/bin/awk /bin/awk
ln -s /usr/bin/basename /bin/basename
ln -s /usr/bin/rpm /bin/rpm
ln -s /usr/lib/i386-linux-gnu/libpthread_nonshared.a /usr/lib/libpthread_nonshared.a
ln -s /usr/lib/i386-linux-gnu/libc_nonshared.a /usr/lib/libc_nonshared.a
ln -s /usr/lib/i386-linux-gnu/libstdc++.so.6 /lib/
ln -s /usr/lib/i386-linux-gnu/libstdc++.so.6 /usr/lib/
ln -s /usr/lib/i386-linux-gnu/libstdc++.so.5 /lib/
ln -s /usr/lib/i386-linux-gnu/libstdc++.so.5 /usr/lib/
echo 'Red Hat Linux release 5' > /etc/redhat-release
```

* 针对64位系统（有所疑惑，后面需要继续测试,我是在/lib, /usr/lib64, /lib64都进行了链接，具体应该是哪个，有待确认，同时，32位的libstdc++.so.5这个文件，死活找不到，是从网上下的）

```
ln -s /etc /etc/rc.d
ln -s /lib/x86_64-linux-gnu/libgcc_s.so.1 /lib64/
ln -s /usr/bin/awk /bin/awk
ln -s /usr/bin/basename /bin/basename
ln -s /usr/bin/rpm /bin/rpmln -s /usr/lib/x86_64-linux-gnu/libpthread_nonshared.a /usr/lib64/libpthread_nonshared.a
ln -s /usr/lib/x86_64-linux-gnu/libc_nonshared.a /usr/lib64/libc_nonshared.a
ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /lib64/
ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /usr/lib64/
ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.5 /lib64/
ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.5 /usr/lib64/
echo 'Red Hat Linux release 5' > /etc/redhat-release
```

## 8. 正式安装

会遇到很多问题，先运行这几个脚本

```
sed -i 's/^\(\s*\$(MK_EMAGENT_NMECTL)\)\s*$/\1 -lnnz11/g' $ORACLE_HOME/sysman/lib/ins_emagent.mk
sed -i 's/^\(TNSLSNR_LINKLINE.*\$(TNSLSNR_OFILES)\) \(\$(LINKTTLIBS)\)/\1 -Wl,--no-as-needed \2/g' $ORACLE_HOME/network/lib/env_network.mk
sed -i 's/^\(ORACLE_LINKLINE.*\$(ORACLE_LINKER)\) \(\$(PL_FLAGS)\)/\1 -Wl,--no-as-needed \2/g' $ORACLE_HOME/rdbms/lib/env_rdbms.mk
sed -i 's/^\(\$LD \$LD_RUNTIME\) \(\$LD_OPT\)/\1 -Wl,--no-as-needed \2/g' $ORACLE_HOME/bin/genorasdksh
sed -i 's/^\(\s*\)\(\$(OCRLIBS_DEFAULT)\)/\1 -Wl,--no-as-needed \2/g' $ORACLE_HOME/srvm/lib/ins_srvm.mk
```

之后 retry应该就可以了。
