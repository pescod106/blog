## CentOS7 firewalld防火墙

**打开某一端口**

1. ```firewall-cmd --zone=public --add-port=80/tcp --permanent```
2. ```firewall-cmd --reload```

[CentOS7中firewalld的使用](https://www.cnblogs.com/moxiaoan/p/5683743.html)
