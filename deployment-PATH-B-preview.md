
================================================
部署详情
================================================

#### Installation Path B installs Cloudera Manager using packages downloaded from a repository. There are several options for installing the JDK, Agents, CDH, and Managed Service packages:


>* Install these items manually using packages. You can use utilities such as Puppet or Chef to help with the installation of these items across all the hosts in a cluster.

>* Cloudera Manager can install them for you on all of the hosts in your cluster. If you choose Cloudera Manager installation, you can select installation using packages or Cloudera Manager parcels(**we choice**).

#### In order for Cloudera Manager to automate installation of Cloudera Manager Agent packages or CDH and managed service software, cluster hosts must satisfy the following requirements:

>* Allow the Cloudera Manager Server host to **have uniform SSH access on the same port to all hosts**. See Networking and Security Requirements for further information.

>* All hosts must have access to standard package repositories and either archive.cloudera.com or a local repository with the required installation files.



====================================================
资源需求及系统配置
====================================================

Cloudera Manager requires the following resources:

------------------
Disk Space
------------------

**Cloudera Manager Server**

* 5 GB on the partition hosting /var.

* 500 MB on the partition hosting /usr.

* For parcels, the space required depends on the number of parcels you download to the Cloudera Manager Server and distribute to Agent 
hosts. You can download multiple parcels of the same product, of different versions and different builds. If you are managing multiple 
clusters, only one parcel of a product/version/build/distribution is downloaded on the Cloudera Manager Server—not one per cluster. 
In the local parcel repository on the Cloudera Manager Server, the approximate sizes of the various parcels are as follows:

>* CDH 5 (which includes Impala and Search) - 1.5 GB per parcel (packed), 2 GB per parcel (unpacked)

>* Impala - 200 MB per parcel

>* Cloudera Search - 400 MB per parcel

>* Cloudera Management Service -The Host Monitor and Service Monitor databases are stored on the partition hosting /var. Ensure that you have at least 20 GB available on this partition.

>* Agents - On Agent hosts, each unpacked parcel requires about three times the space of the downloaded parcel on the Cloudera Manager

>* Server. By default, unpacked parcels are located in /opt/cloudera/parcels.

---------
RAM 
--------
4 GB is recommended for most cases and is required when using Oracle databases. 2 GB might be sufficient for non-Oracle deployments with fewer than 100 hosts. However, to run the Cloudera Manager Server on a machine with 2 GB of RAM, you must tune down its maximum  heap size (by modifying -Xmx in /etc/default/cloudera-scm-server). Otherwise the kernel might kill the Server for consuming too much RAM.

--------
Python
---------
Cloudera Manager requires Python 2.4 or higher, but Hue in CDH 5 and package installs of CDH 5 require Python 2.6 or 2.7. All
supported operating systems include Python version 2.4 or higher.

-------
Perl
-----------
Cloudera Manager requires perl.


==============================================================
CDH and Cloudera Manager Networking and Security Requirements
==============================================================

The hosts in a Cloudera Manager deployment must satisfy the following networking and security requirements:


-----------------------
* CDH requires IPv4. IPv6 is not supported and must be disabled.


* Multihoming CDH or Cloudera Manager is not supported outside specifically certified Cloudera partner appliances. Cloudera finds that current Hadoop architectures combined with modern network infrastructures and security practices remove the need for multihoming. 
Multihoming, however, is beneficial internally in appliance form factors to take advantage of high-bandwidth InfiniBand interconnects.
 Although some subareas of the product might work with unsupported custom multihoming configurations, there are known issues with
multihoming. In addition, unknown issues can arise because multihoming is not covered by the test matrix outside the Cloudera-certified
partner appliances.

* Cluster hosts must have a working network name resolution system and correctly formatted /etc/hosts file. All cluster hosts must have properly configured forward and reverse host resolution through DNS. The /etc/hosts files must:

Contain consistent information about hostnames and IP addresses across all hosts
>* Not contain uppercase hostnames
>* Not contain duplicate IP addresses
>* Cluster hosts must not use aliases, either in /etc/hosts or in configuring DNS.
>* In most cases, the Cloudera Manager Server must have SSH access to the cluster hosts when you run the installation or upgrade wizard. You must log in using a root account or an account that has password-less sudo permission. For authentication during the installation and upgrade procedures, you must either enter the password or upload a public and private key pair for the root or sudo user account. If you want to use a public and private key pair, the public key must be installed on the cluster hosts before you use Cloudera Manager.Cloudera Manager uses SSH only during the initial install or upgrade. Once the cluster is set up, you can disable root SSH access or change the root password. Cloudera Manager does not save SSH credentials, and all credential information is discarded when the installation is complete.

* If single user mode is not enabled, the Cloudera Manager Agent runs as root so that it can make sure the required directories are created and that processes and files are owned by the appropriate user (for example, the hdfs and mapred users).

* No blocking is done by Security-Enhanced Linux (SELinux).

* Cloudera Manager can install them for you on all of the hosts in your cluster. If you choose Cloudera Manager installation, you can select installation using packages or Cloudera Manager parcels. In order for Cloudera Manager to automate installation of Cloudera Manager Agent packages or CDH and managed service software, cluster hosts must satisfy the following requirements:

* Allow the Cloudera Manager Server host to have uniform SSH access on the same port to all hosts. See Networking and Security Requirements for further information.

* All hosts must have access to standard package repositories and either archive.cloudera.com or a local repository with the required installation files.
