==================
MySQL Database
==================

To use a MySQL database, follow these procedures.

-----------------
1. Installing the MySQL Server

2. Configuring and Starting the MySQL Server

3. Installing the MySQL JDBC Driver

4. Creating Databases for Activity Monitor, Reports Manager, Hive Metastore Server, Sentry Server, Cloudera Navigator Audit Server, and Cloudera Navigator Metadata Server

5. Configuring the Hue Server to Store Data in MySQL

6. Configuring MySQL for Oozie

---------------------------


 
###Installing the MySQL Server

```html
Note:
If you already have a MySQL database set up, you can skip to the section Configuring and Starting the MySQL Server to verify that your MySQL configurations meet the requirements for Cloudera Manager.

It is important that the datadir directory, which, by default, is /var/lib/mysql, is on a partition that has sufficient free space.

Cloudera Manager installation fails if GTID-based replication is enabled in MySQL.
```




1.  Install the MySQL database. 

    `$ sudo yum install mysql-server `




2. Configuring and Starting the MySQL Server,Stop the MySQL server if it is running. 

    `$ sudo service mysqld stop `




3. Move old InnoDB log files /var/lib/mysql/ib_logfile0 and /var/lib/mysql/ib_logfile1 out of /var/lib/mysql/ to a backup location.




4. Determine the location of the option file, my.cnf.


5. Update my.cnf so that it conforms to the following requirements: 

>* To prevent deadlocks, set the isolation level to **read committed**.


>* **Configure the InnoDB engine**. Cloudera Manager will not start if its tables are configured with the MyISAM engine. (Typically, tables revert to MyISAM if the InnoDB engine is misconfigured.) To check which engine your tables are using, run the following command from the MySQL shell: mysql> show table status;


>* The default settings in the MySQL installations in most distributions use conservative buffer sizes and memory usage. Cloudera Management Service roles need high write throughput because they might insert many records in the database. Cloudera recommends that you **set the innodb_flush_method property to O_DIRECT**.


>* Set the max_connections property according to the size of your cluster: 


>> ◾Small clusters (fewer than 50 hosts) - You can store more than one database (for example, both the Activity Monitor and Service Monitor) on the same host. If you do this, you should: ◾Put each database on its own storage volume.


>> ◾Allow 100 maximum connections for each database and then add 50 extra connections. For example, for two databases, set the maximum connections to 250. If you store five databases on one host (the databases for Cloudera Manager Server, Activity Monitor, Reports Manager, Cloudera Navigator, and Hive metastore), set the maximum connections to 550.


>> ◾Large clusters (more than 50 hosts) - Do not store more than one database on the same host. Use a separate host for each database/host pair. The hosts need not be reserved exclusively for databases, but each database should be on a separate host.



>* Binary logging is not a requirement for Cloudera Manager installations. Binary logging provides benefits such as MySQL replication or point-in-time incremental recovery after database restore. Examples of this configuration follow. For more information, see The Binary Log.



Here is an option file with Cloudera recommended settings:

---------------------
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
----------------


6.If AppArmor is running on the host where MySQL is installed, you might need to configure AppArmor to allow MySQL to write to the binary.

7.Ensure the MySQL server starts at boot. 

    `$ sudo /sbin/chkconfig mysqld on `
    `$ sudo /sbin/chkconfig --list mysqld mysqld 0:off 1:off 2:on 3:on 4:on 5:on 6:off `

8.Start the MySQL server: 

    `$ sudo service mysqld start `



9.Set the MySQL root password. In the following example, the current root password is blank. Press the Enter key when you're prompted for the root password. 

```html
$ sudo /usr/bin/mysql_secure_installation
[...]
Enter current password for root (enter for none):
OK, successfully used password, moving on...
[...]
Set root password? [Y/n] y
New password:
Re-enter new password:
Remove anonymous users? [Y/n] Y
[...]
Disallow root login remotely? [Y/n] N
[...]
Remove test database and access to it [Y/n] Y
[...]
Reload privilege tables now? [Y/n] Y
All done!
```


 
###Installing the MySQL JDBC Driver


Install the JDBC driver on the Cloudera Manager Server host, as well as hosts to which you assign the Activity Monitor, Reports Manager, Hive Metastore Server, Sentry Server, Cloudera Navigator Audit Server, and Cloudera Navigator Metadata Server roles.

```html
Note: If you already have the JDBC driver installed on the hosts that need it, you can skip this section. However, MySQL 5.6 requires a driver version 5.1.26 or higher.
````

**Cloudera recommends that you assign all roles that require databases on the same host and install the driver on that host**. Locating all such roles on the same host is recommended but not required. If you install a role, such as Activity Monitor, on one host and other roles on a separate host, you would install the JDBC driver on each host running roles that access the database. 


1. Download the MySQL JDBC driver from http://www.mysql.com/downloads/connector/j/5.1.html.

2. Extract the JDBC driver JAR file from the downloaded file. 

   For example: `tar zxvf mysql-connector-java-5.1.31.tar.gz`
 
3. Copy the JDBC driver, renamed, to the relevant host. For example:

   `$ sudo cp mysql-connector-java-5.1.31/mysql-connector-java-5.1.31-bin.jar /usr/share/java/mysql-connector-java.jar`

   If the target directory does not yet exist on this host, you can create it before copying the JAR file. For example:

   ` sudo mkdir -p /usr/share/java/`

   `$ sudo cp mysql-connector-java-5.1.31/mysql-connector-java-5.1.31-bin.jar /usr/share/java/mysql-connector-java.jar`



```html
ote: Do not use the yum install command to install the MySQL driver package, because it installs openJDK, and then uses the Linux alternatives command to set the system JDK to be openJDK.
```




###Return to Establish Your Cloudera Manager Repository Strategy.

 
Creating Databases for Activity Monitor, Reports Manager, Hive Metastore Server, Sentry Server, Cloudera Navigator Audit Server, and Cloudera Navigator Metadata Server


Create databases and user accounts for components that require databases: 
 
>* If you are not using the Cloudera Manager installer, the Cloudera Manager Server.
>* Cloudera Management Service roles: ◦Activity Monitor (if using the MapReduce service)
>* Reports Manager
>* Each Hive metastore
>* Sentry Server
>* Cloudera Navigator Audit Server
>* Cloudera Navigator Metadata Server

You can create these databases on the host where the Cloudera Manager Server will run, or on any other hosts in the cluster. For performance reasons, you should install each database on the host on which the service runs, as determined by the roles you assign during installation or upgrade. In larger deployments or in cases where database administrators are managing the databases the services use, you can separate databases from services, but use caution.

*The database must be configured to support UTF-8 character set encoding.*

Record the values you enter for database names, usernames, and passwords. The Cloudera Manager installation wizard requires this information to correctly connect to these databases.

 1.Log into MySQL as the root user: 
 `$ mysql -u root -p`
 `Enter password:`

2.Create databases for the Activity Monitor, Reports Manager, Hive Metastore Server, Sentry Server, Cloudera Navigator Audit Server, and Cloudera Navigator Metadata Server:

`mysql> create database database DEFAULT CHARACTER SET utf8;`
`Query OK, 1 row affected (0.00 sec)`

`mysql> grant all on database.* TO 'user'@'%' IDENTIFIED BY 'password';`
`Query OK, 0 rows affected (0.00 sec)`

database, user, and password can be any value. The examples match the default names provided in the Cloudera Manager configuration settings: 
<table>
  <tr><td>Role</td><td>Database</td><td>User</td><td>Password</td></tr>
 <tr><td>Activity Monitor</td><td>amon</td><td>amon</td><td>amon_password</td></tr>
 <tr><td>Reports Manager</td><td>rman</td><td>rman</td><td>rman_password</td></tr>
 <tr><td>Hive Metastore Server</td><td>hive</td><td>hive</td><td>hive_password</td></tr>
 <tr><td>Sentry Server</td><td>sentry</td><td>sentry</td><td>sentry_password</td></tr>
 <tr><td>Cloudera Navigator Audit Server</td><td>nav</td><td>nav</td><td>nav_password</td></tr>
 <tr><td>Cloudera Navigator Metadata Server</td><td>navms</td><td>navms</td><td>navms_password</td></tr>
</table>





 
###Configuring the Hue Server to Store Data in MySQL

```html
Note: Cloudera recommends InnoDB over MyISAM as the Hue MySQL engine. On CDH 5, Hue requires InnoDB.
```

For information about installing and configuring a MySQL database , see MySQL Database.

> 1.In the Cloudera Manager Admin Console, go to the Hue service status page.
> 2.Select Actions > Stop. Confirm you want to stop the service by clicking Stop.
> 3.Select Actions > Dump Database. Confirm you want to dump the database by clicking Dump Database.
> 4.Note the host to which the dump was written under Step in the Dump Database Command window. You can also find it by selecting Commands > Recent Commands > Dump Database.
> 5.Open a terminal window for the host and go to the dump file in /tmp/hue_database_dump.json.
> 6.Remove all JSON objects with useradmin.userprofile in the model field, for example:

```html
 {
   "pk": 14,
   "model": "useradmin.userprofile",
   "fields":
   { "creation_method": "EXTERNAL", "user": 14, "home_directory": "/user/tuser2" }
   },
   ```

> 7.Set strict mode in /etc/my.cnf and restart MySQL: 

 ` [mysqld]`
 `sql_mode=STRICT_ALL_TABLES`

> 8.Create a new database and grant privileges to a Hue user to manage this database. For example:

  ` mysql> create database hue;`
  ` Query OK, 1 row affected (0.01 sec)`
  `mysql> grant all on hue.* to 'hue'@'localhost' identified by 'secretpassword';`
  ` Query OK, 0 rows affected (0.00 sec)`

> 9.In the Cloudera Manager Admin Console, click the Hue service.
> 10.Click the Configuration tab.
> 11.Select Scope > All.
> 12.Select Category > Database.
> 13.Specify the settings for Hue Database Type, Hue Database Hostname, Hue Database Port, Hue Database Username, Hue Database Password, and Hue Database Name. For example, for a MySQL database on the local host, you might use the following values: 

◦Hue Database Type = mysql
◦Hue Database Hostname = host
◦Hue Database Port = 3306
◦Hue Database Username = hue
◦Hue Database Password = secretpassword
◦Hue Database Name = hue

> 14.Optionally restore the Hue data to the new database: 
a.Select Actions > Synchronize Database.
b.Determine the foreign key ID. 
  `$ mysql -uhue -psecretpassword`
  `mysql > SHOW CREATE TABLE auth_permission;`

c.(InnoDB only) Drop the foreign key that you retrieved in the previous step.
  `mysql > ALTER TABLE auth_permission DROP FOREIGN KEY content_type_id_refs_id_XXXXXX;`

d.Delete the rows in the django_content_type table. 
  `mysql > DELETE FROM hue.django_content_type;`

e.In Hue service instance page, click Actions > Load Database. Confirm you want to load the database by clicking Load Database.

f.(InnoDB only) Add back the foreign key. 
  `mysql > ALTER TABLE auth_permission ADD FOREIGN KEY (content_type_id) REFERENCES django_content_type (id);`


> 15.Start the Hue service.

 
###Configuring MySQL for Oozie


 
Install and Start MySQL 5.x

 See MySQL Database.

```html
Create the Oozie Database and Oozie MySQL User

For example, using the MySQL mysql command-line tool:
$ mysql -u root -p
Enter password: ******

mysql> create database oozie;
Query OK, 1 row affected (0.03 sec)

mysql>  grant all privileges on oozie.* to 'oozie'@'localhost' identified by 'oozie';
Query OK, 0 rows affected (0.03 sec)

mysql>  grant all privileges on oozie.* to 'oozie'@'%' identified by 'oozie';
Query OK, 0 rows affected (0.03 sec)

mysql> exit
Bye```

 
Add the MySQL JDBC Driver JAR to Oozie

Copy or symbolically link the MySQL JDBC driver JAR into one of the following directories:
•For installations that use packages: /var/lib/oozie/
•For installations that use parcels: /opt/cloudera/parcels/CDH/lib/oozie/lib/
directory. 
Note: You must manually download the MySQL JDBC driver JAR file.
