=========
数据库准备
=========
1.安装mysql

	`yum install mysql-server`

2.配置mysql配置文件 /etc/my.cnf

```html

	[mysqld]
	transaction-isolation = READ-COMMITTED
	# Disabling symbolic-links is recommended to prevent assorted security risks;
	# to do so, uncomment this line:
	# symbolic-links = 0
	
	key_buffer = 16M
	key_buffer_size = 32M
	max_allowed_packet = 32M
	thread_stack = 256K
	thread_cache_size = 64
	query_cache_limit = 8M
	query_cache_size = 64M
	query_cache_type = 1
	
	max_connections = 550
	#expire_logs_days = 10
	#max_binlog_size = 100M
	
	#log_bin should be on a disk with enough free space. Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your system
	#and chown the specified folder to the mysql user.
	log_bin=/var/lib/mysql/mysql_binary_log
	
	# For MySQL version 5.1.8 or later. Comment out binlog_format for older versions.
	binlog_format = mixed
	
	read_buffer_size = 2M
	read_rnd_buffer_size = 16M
	sort_buffer_size = 8M
	join_buffer_size = 8M
	
	# InnoDB settings
	innodb_file_per_table = 1
	innodb_flush_log_at_trx_commit  = 2
	innodb_log_buffer_size = 64M
	innodb_buffer_pool_size = 4G
	innodb_thread_concurrency = 8
	innodb_flush_method = O_DIRECT
	innodb_log_file_size = 512M
	[mysqld_safe]
	log-error=/var/log/mysqld.log
	pid-file=/var/run/mysqld/mysqld.pid
	

sql_mode=STRICT_ALL_TABLES
```

3.启动mysql，并设置开机启动

	`service mysqld start`
	`/sbin/chkconfig mysqld on`
	`/sbin/chkconfig --list mysqld`


4.初始化设置mysql,设置密码

	`/usr/bin/mysql_secure_installation`

5.配置jdbc,下载自<http://www.mysql.com/downloads/connector/j/5.1.html>，解压

	`mkdir  /usr/share/java`
	
	`cp mysql-connector-java-5.1.34.jar  /usr/share/java/mysql-connector-java.jar`
	

6.配置 mysql中cm server 账号密码(该步骤在安装cm server 结束后进行)

	`mysql> grant all on *.* to 'temp'@'%' identified by 'temp' with grant option;`
	
	`/usr/share/cmf/schema/scm_prepare_database.sh mysql -h myhost1.sf.cloudera.com -utemp -ptemp --scm-host myhost2.sf.cloudera.com scm scm scm`



==============
本地源配置
============


1. 安装必要工具

	`yum install httpd createrepo`
	
2.启动appache服务

	`service httpd start`
	
3.下载cm文件 <http://archive.cloudera.com/cm5/repo-as-tarball/>

4.解压文件
	`tar xvfz cm5.0.0-centos6.tar.gz`
	`mv cm /var/www/html`
	`chmod -R ugo+rX /var/www/html/cm`

5.配置repo文件 /etc/yum.repos.d/myrepo.repo
	```html
	[myrepo]
	name=myrepo
	baseurl=http://hostname/5.5.0/
	enabled=1
	gpgcheck=0
```

6.更新源

	`yum update`



=========
安装cm
========

1.安装jdk

	`yum install oracle-j2sdk1.7`
	
2.安装cm server

	`yum install cloudera-manager-daemons cloudera-manager-server`





