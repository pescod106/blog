## MySql安装
### 1.配置yum源
在MySQL官网下载yum源rpm安装包*http://dev.mysql.com/downloads/repo/yum/*
![](https://www.ltar.com/wp-content/uploads/2018/05/mysql_yum_rpm.png)
#### A.下载mysql源安装包


```
wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm

```


#### B.安装MySQL源


```
yum localinstall mysql57-community-release-el7-8.noarch.rpm

```


#### C.检查MySQL源是否安装成功

```
yum repolist enabled | grep "mysql.*-community.*"
```


![](https://www.ltar.com/wp-content/uploads/2018/05/mysql_repolist.png)

看到上图说明安装成功

可以修改vim /etc/yum.repos.d/mysql-community.repo源，改变默认安装的mysql版本。比如要安装5.6版本，将5.7源的enabled=1改成enabled=0。然后再将5.6源的enabled=0改成enabled=1即可。改完之后的效果如下所示：
![](https://www.ltar.com/wp-content/uploads/2018/05/mysql-repo.png)

### 2.安装MySQL

```
install mysql-community-server
```


### 3.启动MySQL

```
systemctl start mysqld
```


**查看MySQL是否启动**

```
systemctl status mysqld
```

![](https://www.ltar.com/wp-content/uploads/2018/05/mysqld_status.png)

### 4.开机启动

```
systemctl enable mysqld
```



```
systemctl daemon-reload
```

### 5.修改root本地登录密码
mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个默认密码。通过下面的方式找到root默认密码，然后登录mysql进行修改：

```
grep 'temporary password' /var/log/mysqld.log
```

![](https://www.ltar.com/wp-content/uploads/2018/05/mysql_temp_pwd.png)

**登录**


```
mysql -uroot -p
```


**修改密码**


```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```


### 6.添加远程登录账户
默认只允许root帐户在本地登录，如果要在其它机器上连接mysql，必须修改root允许远程连接，或者添加一个允许远程连接的帐户，为了安全起见，我添加一个新的帐户：


```
 GRANT ALL PRIVILEGES ON *.* TO 'newuser'@'%' IDENTIFIED BY 'newpasswd!' WITH GRANT OPTION;
```

### 7.配置默认编码为utf8
修改/etc/my.cnf配置文件，在[mysqld]下添加编码配置，如下所示：
>character_set_server=utf8
>
>init_connect='SET NAMES utf8'

重新启动mysql服务，查看数据库默认编码如下所示：

![](https://www.ltar.com/wp-content/uploads/2018/05/mysql_character.png)

默认配置文件路径：

|文件类型|路径|
|---|---| 
|配置文件|/etc/my.cnf |
|日志文件|/var/log//var/log/mysqld.log |
|服务启动脚本|/usr/lib/systemd/system/mysqld.service |
|socket文件|/var/run/mysqld/mysqld.pid|