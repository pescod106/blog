<div class="article_title">
<h1></h1>
</div>
<div class="article_manage clearfix"></div>
<div class="category clearfix">
<div class="category_r"></div>
</div>
<div id="article_content" class="article_content">
<h2>如何启动/停止/重启MySQL</h2>
<h3>一、启动方式</h3>
<ul>
 	<li>使用 service 启动：service mysqld start</li>
 	<li>使用 mysqld 脚本启动：/etc/inint.d/mysqld start</li>
 	<li>使用 safe_mysqld 启动：safe_mysqld&amp;</li>
</ul>
<h3>二、停止</h3>
<ul>
 	<li>使用 service 启动：service mysqld stop</li>
 	<li>使用 mysqld 脚本启动：/etc/inint.d/mysqld stop</li>
 	<li>mysqladmin shutdown</li>
</ul>
<h3>三、重启</h3>
<ul>
 	<li>使用 service 启动：service mysqld restart</li>
 	<li>使用 mysqld 脚本启动：/etc/inint.d/mysqld restart</li>
</ul>
</div>