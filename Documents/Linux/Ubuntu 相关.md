
1. 开始SSH登陆

```bash
sudo apt-get install openssh-server
```

```bash
sudo /etc/init.d/ssh start
```

或

```bash
sudo service ssh start 
```

ssh-server配置文件位于/etc/ssh/sshd_config，在这里可以定义SSH的服务端口，默认端口是22，你可以自己定义成其他端口号，如222。（或把配置文件中的”PermitRootLogin without-password”加一个”#”号,把它注释掉，再增加一句”PermitRootLogin yes”） 

然后重启SSH服务： 

```bash
sudo /etc/init.d/ssh stop 
sudo /etc/init.d/ssh start
```

2. 开启远程桌面

使用ubuntu自带的桌面共享即可，注意：

```bash
gsettings set org.gnome.Vino require-encryption false
```

3. Ubuntu删除Snap

```bash
sudo apt autoremove --purge snapd
```