
```sql
---客户端/服务器之间通信的缓存区的最大大小
set global max_allowed_packet=10000000000;

---TCP/IP和套接字通信缓冲区大小,创建长度达net_buffer_length的行
set global net_buffer_length=1000000;

---对后续起的交互链接有效
SET GLOBAL interactive_timeout=288000000;

---对当前交互链接有效
SET GLOBAL wait_timeout=28800000;
```

