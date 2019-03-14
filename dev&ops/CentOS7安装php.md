## PHP安装
### 1.通过yum安装php,php-fpm,php-mysql
```yum install php php-fpm php-mysql```

### 2.启动php-fpm
```service php-fpm start```

### 3.查看php-fpm是否启动，查看9000端口是否启动
![](http://cdn.ltar.com/wp-content/uploads/2018/05/php_netstat.png)

### 4.设置php-fpm开机启动
```chkconfig php-fpm on```