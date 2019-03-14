# Linux上安装WordPress
## 一、WordPress介绍
* [WordPress](https://wordpress.org/)是一个基于PHP的开源博客系统。它起源于2003年，是目前世界上最流行的开源博客系统。
* WordPress可以搭建功能强大的网络信息发布平台，但更多的是应用于个性化的博客。

## 二、准备工作
* CentOS
* [LNMP环境](http://39.105.67.180/2018/05/21/54309400-5cca-4cf6-8d40-360dddaaaee9/)

## 三、安装

#### 1.使用wget下载
```wget https://wordpress.org/latest.zip```
#### 2.解压
* 移动到指定目录```mv latest.zip /opt/php_web_site/```
* 解压生成wordpress目录```unzip latest.zip```

## 四、WordPress配置

### 1.数据库配置
#### A.进入MySQL
```mysql -u root -p```

#### B.创建wordpress数据库
```create database wordpress;```

#### C.为WordPress创建一个新用户
```create user wordpress@localhost identified by 'wordpresspwd';```

#### D.设置权限，只可以操作本数据库(本数据库的所有权限)
```grant all privileges on wordpress.* to wordpress@localhost identified by 'wordpresspwd';```

#### E.将权限生效
```flush privileges;```

### 2.配置WordPress连接数据库信息
1. 登录服务器进入wordpress目录
2. 创建wp-config.php文件```cp wp-config-sample.php wp-config.php```
3. 填写数据库信息```vim wp-config.php```

```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpress');

/** MySQL database password */
define('DB_PASSWORD', 'wordpresspwd');

/** MySQL hostname */
define('DB_HOST', 'localhost');
```

### 开启80端口
#### 1.服务器开启80端口
```firewall-cmd --zone=public --add-port=80/tcp --permanent```

```firewall-cmd --reload```
#### 2.阿里云服务器开启80端口
![]()

### 配置Nginx

#### 1.添加wordpress配置文件

##### A.创建目录
```mkdir sites-available```,```mkdir sites-enabled```

##### B.创建配置文件
```cd sites-available```

```vim wordpress.conf```
添加如下内容

```
server {
 listen 80;
 error_log  /var/log/nginx/www.domain.com/error.log warn;
 access_log /var/log/nginx/www.domain.com/access.log ;
 server_name 39.105.67.180;
 index index.html index.htm index.php;
 root /opt/php_web_site/wordpress;
 location / {
   try_files $uri $uri/ /index.php$args;
 }

 location ~ \.php$ {
  fastcgi_pass 127.0.0.1:9000;
  ## With php5-fpm:
  #fastcgi_pass unix:/var/run/php5-fpm.sock;
  fastcgi_index index.php;
  fastcgi_param SCRIPT_FILENAME /opt/php_web_site/wordpress$fastcgi_script_name;
  include fastcgi_params;
 }
}
```

##### C.引入配置文件

* 进入sites-enabled目录创建软连接```ln -s ../sites-available/wordpress.conf wordpress.conf```
* 在nginx.conf的http块中添加```include sites-enabled/*.conf;```

## 五、WordPress安装
在浏览器中输入域名或者IP进行安装
![]()