## 1.LNMP介绍

LNMP是Linux、Nginx、MySQL、PHP的缩写，是最常见的web服务器的运行环境之一

### 2.Nginx的安装

<a href="http://39.105.67.180/2018/05/22/01085875-bbba-4d11-b6df-2b830f1868b3/" target="_blank" rel="noopener">CentOS7安装Nginx</a>

### 3.MySQL的安装

<a href="http://39.105.67.180/2018/05/22/e48d5ec0-a46b-47c5-aabe-2af0f8e6cd04/" target="_blank" rel="noopener">CentOS7安装MySQL</a>

### 4.PHP的安装

<a href="http://39.105.67.180/2018/05/22/42be8416-a528-4e33-b16c-3d08de5c3236/" target="_blank" rel="noopener">CentOS7安装PHP</a>

### 验证LNMP
#### 1.在Nginx的/etc/nginx/conf.d文件夹中创建php.conf文件，设置端口为8000，如下：
通过vim /etc/nginx/conf.d/php.conf打开文件，输入一下内容保存

```
server {
    listen 8000;
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        root           /usr/share/php;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

> Nginx修改配置，需要重启```service nginx reload```或者```service nginx restart```

#### 2.在php的目录/usr/share/php文件夹中创建phpinfo.php文件
通过vim /etc/nginx/conf.d/php.conf打开文件，输入以下内容保存。

<pre>&lt;?php echo phpinfo(); ?&gt;</pre>


#### 3.验证防火墙是否开启，如果请把8000端口加入白名单

<img class="alignleft size-full wp-image-99" src="http://39.105.67.180/wp-content/uploads/2018/05/firewall_status.png" alt="" width="2344" height="762" />

firawalld打开某一端口
```firewall-cmd --zone=public --add-port=80/tcp --permanent```

```firewall-cmd --reload```

#### 4.通过浏览器验证

打开http://&lt;外网IP地址&gt;:8000/phpinfo.php,看是否显示php信息

<img class="alignleft size-full wp-image-100" src="http://39.105.67.180/wp-content/uploads/2018/05/lnmp_chk.png" alt="" width="2000" height="476" />