=============
Required Databases
================

The Cloudera Manager Server, Oozie Server, Sqoop Server, Activity Monitor, Reports Manager, Hive Metastore Server, Sentry Server, Cloudera Navigator Audit Server, and Cloudera Navigator Metadata Server all require databases. The type of data contained in the databases and their estimated sizes are as follows:

* **Cloudera Manager** - Contains all the information about services you have configured and their role assignments, all configuration history, commands, users, and running processes. This relatively small database (<100 MB) is the most important to back up. 

---------------------
######Important: When processes restart, the configuration for each of the services is redeployed using information that is saved in the Cloudera Manager database. If this information is not available, your cluster will not start or function correctly. You must therefore schedule and maintain regular backups of the Cloudera Manager database to recover the cluster in the event of the loss of this database.
--------------------------

* Oozie Server - Contains Oozie workflow, coordinator, and bundle data. Can grow very large.

* Sqoop Server - Contains entities such as the connector, driver, links and jobs. Relatively small.

* Activity Monitor - Contains information about past activities. In large clusters, this database can grow large. Configuring an Activity Monitor database is only necessary if a MapReduce service is deployed.

* Reports Manager - Tracks disk utilization and processing activities over time. Medium-sized.

* **Hive Metastore Server** - Contains Hive metadata. Relatively small.

* **Sentry Server** - Contains authorization metadata. Relatively small.

* Cloudera Navigator Audit Server - Contains auditing information. In large clusters, this database can grow large.

* Cloudera Navigator Metadata Server - Contains authorization, policies, and audit report metadata. Relatively small.

* The Cloudera Manager Service Host Monitor and Service Monitor roles have an internal datastore.

----------
> ######Cloudera Manager supports deploying different types of databases in a single environment, but doing so can create unexpected complications. Cloudera recommends choosing one supported database provider for all of the Cloudera databases.

> ######In most cases, you should install databases and services on the same host. For example, if you create the database for Activity Monitor on myhost1, then you should typically assign the Activity Monitor role to myhost1.

> ######You assign the Activity Monitor and Reports Manager roles in the Cloudera Manager wizard during the installation or upgrade process. After completing the installation or upgrade process, you can also modify role assignments in the Management services pages of Cloudera Manager. Although the database location is changeable, before beginning an installation or upgrade, you should decide which hosts to use. 

> ######The JDBC connector for your database must be installed on the hosts where you assign the Activity Monitor and Reports Manager roles.

> ######You can install the database and services on different hosts.Separating databases from services is more likely in larger deployments and in cases where more sophisticated database administrators choose such a configuration. For example, databases and services might be separated if your environment includes Oracle databases that are managed separately by Oracle database administrators.
------------

