## Nginx的安装

### 1. 通过yum安装Nginx

<code>yum install nginx</code>

### 2. 启动Nginx</h4>

<code>service nginx start</code>

### 3. 查看Nginx是否启动成功

#### A.通过netstat查看80端口是否监听
<img src="http://39.105.67.180/wp-content/uploads/2018/05/QQ20180521-221610@2x-1.png" alt="" width="1418" height="114" class="alignleft size-full wp-image-85" />

### 4.通过chkconfig设置开机启动

```chkconfig nginx on```

### 5.查看Nginx版本

```nginx -V```