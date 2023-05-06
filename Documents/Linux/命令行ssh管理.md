
1.  生成密钥文件

    ```bash
    ssh-keygen -t rsa
    ```

    会在\~/.ssh目录下生成对应的密钥文件，可以设置密钥密码，也可以为空

2.  在终端执行scp远程copy命令

    ```bash
    scp ~/.ssh/id_rsa.pub user@hostname:~/.ssh
    ```

    注意，需要远程有.ssh目录。

3.  将公钥追交到授权key中

    在服务器终端下输入：

    ```bash
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    ```

    如果远程服务器上已经存在"~/.ssh/authorized\_keys"文件，那么需要编辑服务器上"~/.ssh/authorized\_keys"文件，将客户端机器上的"id\_rsa.pub"文件内容追加到"\~/.ssh/authorized\_keys"文件中。

4.  别名管理ssh主机

    ```bash
    vim ~/.ssh/config
    ```

    添加参数

        HostName ipAddress
        User userName
        Port port

    就可以通过ssh hostName来登录了。

