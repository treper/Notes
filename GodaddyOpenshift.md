#####GoDaddy Openshift Notes

metinfo的部署配置用到apache2，并在site-available里配置为默认的目录

apache设置默认web app路径

		research@research:~/software/Django-1.3.7$ sudo vim /etc/apache2/sites-enabled/alipay_python.conf 
		
		<VirtualHost *:80>
		WSGIScriptAlias / /home/research/software/alipay_python/apache/mod.wsgi
		
		<Directory /home/research/software/alipay_python>
		Order deny,allow
		Allow from all
		</Directory>
		</VirtualHost>