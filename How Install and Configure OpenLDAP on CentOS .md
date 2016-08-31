===============
Introduction
===============

**LDAP stands for Lightweight Directory Access Protocol.**

LDAP is a solution to access centrally stored information over network. This centrally stored information is organized in a directory that follows X.500 standard.

The information is stored and organized in a hierarchical manner and the advantage of this approach is that the information can be grouped into containers and clients can access these containers whenever needed.

The OpenLDAP hierarchy is almost similar to the DNS hierarchy.

The following are the two most commonly used objects in OpenLDAP:

```html
cn (common name) – This refers to the leaf entries, which are end objects (for example: users and groups)

dc (domain component) – This refers to one of the container entries in the LDAP hierarchy. If in a setup the LDAP hierarchy is mapped to a DNS hierarchy, typically all DNS domains are referred to as DC objects.

For example, if there is user in the hierarchy sam.thegeekstuff.com, the fully distinguished name of this user is referred as cn=sam, dc=thegeekstuff, dc=com. If you noticed in the FDN (fully distinguished name), a comma is used a separator and not a dot, which is common in DNS.
```

By using the different LDAP entry types, you can setup a hierarchical directory structure. This is the reason why openLDAP is so widely used. You can easily build an openLDAP hierarchy where objects in the other locations are easily referred to without storing them on local servers. This makes OpenLDAP a lightweight directory, especially when compared to other directory servers such as Microsoft’s Active directory.
Now lets see how to setup a single instance of an LDAP server that can be used by multiple clients in your network for authentication.


==========
Install OpenLDAP Packages
===========
On CentOS and RedHat, use yum install as shown below, to install the openldap related packages.

`yum install -y openldap openldap-clients openldap-servers`

You should install the following three packages:
```html
openldap-servers – This is the main LDAP server

openldap-clients – This contains all required LDAP client utilities

openldap – This packages contains the LDAP support libraries
```

LDAP Config Files
```html
config.ldif – The LDAP default configuration is stored under a file in /etc/openldap/slapd.d/cn=config.ldif that is created in the LDIF format. This is the LDAP Input Format (LDIF), a specific format that allows you to enter information in to the LDAP directory.

olcDatabase{2}bdb.ldif – You can also modify the settings like number of connections the server can support, timeouts and other database settings under the file /etc/openldap/slapd.d/cn=config/olcDatabase{2}bdb.ldif. This is the file that also contains the parameters like LDAP root user and the base DN.
```


======
Create olcRootDN Account as Admin
======

It is always recommended to create a dedicated user account first with the full permissions to change information on the LDAP database.

Modify the olcDatabase={2}bdb.ldif file, and change the olcRootDN entry. The following is the default entry.

```html
# grep olcRootDN /etc/openldap/slapd.d/cn=config/olcDatabase={2}bdb.ldif
olcRootDN: cn=Manager,dc=my-domain,dc=com
```

Change the above line to an admin user. In this example, user “ramesh” will be the olcRootDN.

`olcRootDN: cn=ramesh,dc=thegeekstuff,dc=com`


==============
Create olcRootPW Root Password
==============

Now use slappasswd command to create a hash for the root password you want to use. Once the password is generated, open the cn=config.ldif file, include the olcRootPW parameter, and copy the hashed password as shown below.

Execute the following command and specify a password. This will generate the hash for the given password.

```html
# slappasswd
New password: SecretLDAPRootPass2015
Re-enter new password: SecretLDAPRootPass2015
{SSHA}1pgok6qWn24lpBkVreTDboTr81rg4QC6
```

Take the hash output of the above command and add it to the oclRootPW parameter in the config.ldif file as shown below.

```html
# vi /etc/openldap/slapd.d/cn=config.ldif
olcRootPW: {SSHA}1pgok6qWn24lpBkVreTDboTr81rg4QC6
```


======
Create olcSuffix Domain Name
=======

Now setup the olcSuffix and to set the domain that you want. Simply modify the line that starts with olcSuffix in the file olcDatabase={2}bdb.ldif as shown below.
```html
# vi /etc/openldap/slapd.d/cn=config/olcDatabase={2}bdb.ldif
olcSuffix: dc=thegeekstuff,dc=com
```

========
Verify The Configuration Files
==========
Use slaptest command to verify the configuration file as shown below. This should display “testing succeeded” message as shown below.

```html
# slaptest -u
config file testing succeeded
```
You might get the following messages during the above command, which you can ignore for now.

```html
54a39508 ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif"
54a39508 ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={2}bdb.ldif"
```

=====
Start the LDAP Server
=========
Start the ldap server as shown below.

```html
# service slapd start
Checking configuration files for slapd: [WARNING]
config file testing succeeded
Starting slapd:                         [  OK  ]
```

========
Verify the LDAP Search
==========
To verify the ldap server is configured successfully, you can use the below command and verify that the domain entry is present.


```html
# ldapsearch -x -b "dc=thegeekstuff,dc=com"
# extended LDIF
#
# LDAPv3
# base <dc=thegeekstuff,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#
# search result
search: 2
result: 32 No such object
# numResponses: 1
```

========
Base LDAP Structure in base.ldif
============
The use of OU (organizational unit) objects can help you in providing additional structure to the LDAP database. If you are planning on adding in different types of entries, such as users, groups, computers, printers and more to the LDAP directory, it makes it easier to put every entry type into its own container.

To create these OU’s, you can create an initial LDIF file as shown in the below example. In this example, this file allows you to create the base container which is dc=thegeekstuff,dc=com and it creates two organizational units with the names users and groups in that container.
```html
# cat base.ldif
dn: dc=thegeekstuff,dc=com
objectClass: dcObject
objectClass: organization
o: thegeekstuff.com
dc: thegeekstuff
dn: ou=users,dc=thegeekstuff,dc=com
objectClass: organizationalUnit
objectClass: top
ou: users
dn: ou=groups,dc=thegeekstuff,dc=com
objectClass: organizationalUnit
objectClass: top
ou: groups
```

=============
Import Base Structure Using ldapadd
=============
Now we can import the base structure in to the LDAP directory using the ldapadd command as shown below.

```html
# ldapadd -x -W -D "cn=ramesh,dc=thegeekstuff,dc=com" -f base.ldif
Enter LDAP Password:
adding new entry "dc=thegeekstuff,dc=com"
adding new entry "ou=users,dc=thegeekstuff,dc=com"
adding new entry "ou=groups,dc=thegeekstuff,dc=com"
```

=======
Verify the Base Structure using ldapsearch
=========

To verify the OUs are successfully created, use the following ldapsearch command.

```html
# ldapsearch -x -W -D "cn=ramesh,dc=thegeekstuff,dc=com" -b "dc=thegeekstuff,dc=com" "(objectclass=*)"
Enter LDAP Password:
```

The output of the above command will display all the objects in the LDAP directory structure.

```html
# extended LDIF
#
# LDAPv3
# base <dc=thegeekstuff,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#
# thegeekstuff.com
dn: dc=thegeekstuff,dc=com
objectClass: dcObject
objectClass: organization
o: thegeekstuff.com
dc: thegeekstuff
# users, thegeekstuff.com
dn: ou=users,dc=thegeekstuff,dc=com
objectClass: organizationalUnit
objectClass: top
ou: users
# groups, thegeekstuff.com
dn: ou=groups,dc=thegeekstuff,dc=com
objectClass: organizationalUnit
objectClass: top
ou: groups
# search result
search: 2
result: 0 Success
# numResponses: 4
# numEntries: 3
```
In the next OpenLDAP article, we’ll explain how to add new users and groups to the LDAP Directory.
