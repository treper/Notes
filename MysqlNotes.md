#####Mysql Common Practice


Mysql异常com.mysql.jdbc.exceptions.jdbc4.CommunicationsException:Communications link failure  

2013-03-27 10:34:06|  分类： 数据库 |举报|字号 订阅
org.springframework.transaction.CannotCreateTransactionException: Could not open JDBC Connection for transaction; nested exception is com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet successfully received from the server was 6,388 milliseconds ago.  The last packet sent successfully to the server was 1,504 milliseconds ago.
        at org.springframework.jdbc.datasource.DataSourceTransactionManager.doBegin(DataSourceTransactionManager.java:240)

主要原因：上述问题是由mysql5数据库的配置引起的。mysql5将其连接的等待时间(wait_timeout)缺省为8小时。在其客户程序中可以这样来查看其值： 

mysql﹥ 

mysql﹥ show global variables like 'wait_timeout'; 

+---------------+---------+ 

| Variable_name | Value | 

+---------------+---------+ 

| wait_timeout | 28800 | 

+---------------+---------+ 

1 row in set (0.00 sec) 


28800 seconds，也就是8小时。 

如果在wait_timeout秒期间内，数据库连接(java.sql.Connection)一直处于等待状态，mysql5就将该连接关 闭。这时，你的Java应用的连接池仍然合法地持有该连接的引用。当用该连接来进行数据库操作时，就碰到上述错误。

解决办法（优先尝试第三种方案）：
（1）在jdbc连接url的配置中，你可以附上“autoReconnect=true”，但这仅对mysql5以前的版本起作用。

（2）既然问题是由mysql5的全局变量wait_timeout的缺省值太小引起的，我们将其改大就好了。 
查看mysql5的手册，发现对wait_timeout的最大值分别是24天/365天(windows/linux)。以windows为 例，假设我们要将其设为21天，我们只要修改mysql5的配置文件“my.ini”(mysql5 installation dir)，增加一行：wait_timeout=1814400 ，需要重新启动mysql5。 
linux系统配置文件：/etc/my.cnf 

（3）我们可以将数据库连接池的 validateQuery、testOnBorrow(testOnReturn)打开，这样在 每次从连接池中取出且准备使用之前(或者使用完且在放入连接池之前)先测试下当前使用是否好用，如果不好用，系统就会自动destory掉。
或者testWhileIdle项是设置是否让后台线程定时检查连接池中连接的可用性。



#####MySQL访问权限设置


指定用户访问，设置访问密码，指定访问主机。

* 设置访问单个数据库权限

		mysql>grant all privileges on test.* to 'root'@'%';（说明：设置用户名为root，密码为空，可访问数据库test）
 
* 设置访问全部数据库权限

		mysql>grant all privileges on *.* to 'root'@'%';（说明：设置用户名为root，密码为空，可访问所有数据库*）

* 设置指定用户名访问权限

		mysql>grant all privileges on *.* to 'lanping'@'%';（说明：设置指定用户名为 lanping ，密码为空，可访问所有数据库*）

* 设置密码访问权限

		mysql>grant all privileges on *.* to 'lanping'@'%' IDENTIFIED BY ' lanping';（说明：设置指定用户名为 lanping ，密码为 lanping ，可访问所有数据库*）

* 设置指定可访问主机权限


		mysql>grant all privileges on *.* to ' lanping '@'10.2.1.11';说明：设置指定用户名为 lanping ，可访问所有数据库*，只有127.0.0.1这台机器有权限访问


#####Navicat无法访问虚拟机里的mysql

注释/etc/mysql/my.cnf里的

	bind-address           = 127.0.0.1

一行并重启mysql

设置访问权限

	grant all privileges on *.* to 'root'@'172.16.1.100' identified by '123456' 