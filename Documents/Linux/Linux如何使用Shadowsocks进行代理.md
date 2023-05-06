
## 安装shadowsocks

```bash
yum install epel-release -y
yum install python-pip
pip install --upgrade pip
pip install shadowsocks
```

安装完成后，配置shadowsocks，举例：

```bash
vi /etc/shadowsocks.json
```

配置文件如下：

```json
{
    "server":"服务器ip地址",
    #ss服务器IP
 
    "server_port":服务器端口,
    #端口
 
    "local_address": "127.0.0.1",
    #本地ip
 
    "local_port":1080,
    #本地端口
 
    "password":"连接代理需要的密码",
    #连接ss密码
 
    "timeout":300,
    #等待超时
 
    "method":"加密方式",
    #加密方式，比如aes-256-cfb
 
    "fast_open": false,
    # true 或 false。如果你的服务器 Linux 内核在3.7+，可以开启 fast_open 以降低延迟。开启方法： echo 3 > /proc/sys/net/ipv4/tcp_fastopen 开启之后，将 fast_open 的配置设置为 true 即可
 
    "workers": 1
    # 工作线程数
}
```

修改完成后，可以通过运行如下命令启动：

```bash
nohup sslocal -c /etc/shadowsocks.json /dev/null 2>&1 &
```

### 可能问题

在shadowsocks的nohup.out中，看到如下错误：

```bash
"AttributeError: /lib64/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup"
```

找到报错的文件

```bash
vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py
```

全文搜索cleanup

将所有
EVP\_CIPHER\_CTX\_cleanup
替换成为
EVP\_CIPHER\_CTX\_reset

## 安装Privoxy

```bash
yum install -y privoxy
```

然后修改配置

```bash
vim /etc/privoxy/config
```

确保如下内容被开启

```bash
forward-socks5t   /               127.0.0.1:1080 .
```

注意，这里后面的ip端口和配置的shadowsocks本地ip和端口呼应

然后启动：

    systemctl start privoxy

## 修改环境变量

```bash
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export ftp_proxy=http://127.0.0.1:8118
```

