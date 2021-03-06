=======================================================================================================================================
Cloudera Manager Deployment . A Cloudera Manager deployment consists of the following software components:

      Oracle JDK
      
      Cloudera Manager Server and Agent packages
      
      Supporting database software
      
      CDH and managed service software
=======================================================================================================================================
      

Demonstration and proof of concept deployments 
==============================================


Installation Path A 
-------------------------

> Automated Installation by Cloudera Manager (**Non-Production Mode**)

Cloudera Manager automates the installation of the Oracle JDK, Cloudera Manager Server, embedded PostgreSQL database, Cloudera Manager Agent, CDH, and managed service software on cluster hosts. Cloudera Manager also configures databases for the Cloudera Manager Server and Hive Metastore and optionally for Cloudera Management Service roles. **This path is recommended for demonstration and proof-of-concept deployments, but is not recommended for production deployments because its not intended to scale and may require database migration as your cluster grows**. To use this method, server and cluster hosts must satisfy the following requirements:
Provide the ability to log in to the Cloudera Manager Server host using a root account or an account that has password-less sudo permission.

 >* Allow the Cloudera Manager Server host to have uniform SSH access on the same port to all hosts. See Networking and Security Requirements for further information.
 >* All hosts must have access to standard package repositories and either archive.cloudera.com or a local repository with the required installation files.

Installation Path B
-----------------------------

> Installation Using Cloudera Manager Parcels or Packages.

you install the Oracle JDK, Cloudera Manager Server, and embedded PostgreSQL database packages on the Cloudera Manager Server host. You have two options for installing Oracle JDK, Cloudera Manager Agent, CDH, and managed service software on cluster hosts: manually install it yourself or use Cloudera Manager to automate installation. In order for Cloudera Manager to automate installation of Cloudera Manager Agent packages or CDH and managed service software, cluster hosts must satisfy the following requirements:

>* Allow the Cloudera Manager Server host to have uniform SSH access on the same port to all hosts. See Networking and Security Requirements for further information.
>* All hosts must have access to standard package repositories and either archive.cloudera.com or a local repository with the required installation files.




Production deployments
===========================

require you to first**manually install and configure a production database for the Cloudera Manager Server and Hive Metastore**. There are two installation options:

Installation Path B
------------

> Installation Using Cloudera Manager Parcels or Packages

you install the Oracle JDK and Cloudera Manager Server packages on the Cloudera Manager Server host. You have two options for installing Oracle JDK, Cloudera Manager Agent, CDH, and managed service software on cluster hosts: manually install it yourself or use Cloudera Manager to automate installation.
In order for Cloudera Manager to automate installation of Cloudera Manager Agent packages or CDH and managed service software, cluster hosts must satisfy the following requirements:
>* Allow the Cloudera Manager Server host to have uniform SSH access on the same port to all hosts. See Networking and Security Requirements for further information.
>* All hosts must have access to standard package repositories and either archive.cloudera.com or a local repository with the required installation files.


Installation Path C
-----------------------

>Manual Installation Using Cloudera Manager Tarballs 

you install the Oracle JDK, Cloudera Manager Server, and Cloudera Manager Agent software using tarballs and use Cloudera Manager to automate installation of CDH and managed service software as parcels.



