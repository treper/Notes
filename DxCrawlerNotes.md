###读秀爬虫项目文档





调试最好使用xwindows图形界面，这样可以看到模拟点击浏览器的行为以及图片到底有没有下载

####数据库设计

#####Mybatis

####基于Qtwebkit的图片爬虫

####部署
#####依赖安装

由于centos的qtwebkit版本滞后，有些函数没有，请使用ubuntu来部署本系统

依赖库:

	sudo apt-get install vim openssh-server build-essential python-dev mysql-server mysql-client libmysqlclient-dev python-setuptools xvfb python-qt4 git libboost-all-dev

python模块:
	
	sudo pip install reportlab pillow python-mysql elixir mechanize MySQL-python

#####建立并导入数据库

建立

	create database piracyfinder

导入

	mysql -uroot -p123456 piracyfinder < proxyplugin_http_proxy.sql
