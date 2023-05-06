
## 配置

```json
{
    "server":"192.168.193.138",
    "server_port":5678,
    "local_address": "127.0.0.1",
    "local_port":5678,
    "password":"Adir@1234",
    "timeout":300,
    "method":"rc4-md5"
}
```

## 启动

申请root后

```bash
ssserver -c /etc/shadowsocks.json -d start
```

