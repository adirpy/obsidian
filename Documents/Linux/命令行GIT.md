
1.  安装git

    ```bash
    sudo apt-get install git
    ```

2.  配置用户名和邮箱

    ```bash
    git config --global user.name "用户名"
    git config --global user.email "邮箱"
    ```

3.  生成ssh公匙、私匙

    ```bash
    ssh-keygen -t rsa
    ```

4.  三次回车

5.  查看公匙进入.sshcd \~/.ssh

6.  打开公匙文件

7.  复制公匙到github登录github-> Accounting settings图标-> SSH key-> Add SSH key-> 填写SSH key的名称（可以起一个自己容易区分的），然后将拷贝的\~/.ssh/id\_rsa.pub文件内容粘帖-> add key”按钮添加。

8.  测试git

    ```bash
    ssh -T git@github.com
    ```

