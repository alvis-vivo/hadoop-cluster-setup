===============
Hadoop配置LDAP集成Kerberos
================

环境说明：

```html
操作系统：Centos6.6
OpenLDAP 版本：2.4.39
Kerberos 版本：1.10.3
CDH 版本：cdh-5.2.0
```

###1.安装服务端

####1.1 安装
同安装 kerberos 一样，这里使用 cdh1 作为服务端安装 openldap。
```html
$ yum install db4 db4-utils db4-devel cyrus-sasl* krb5-server-ldap -y
$ yum install openldap openldap-servers openldap-clients openldap-devel compat-openldap -y
```

查看安装的版本：

```html
$ rpm -qa openldap
openldap-2.4.39-8.el6.x86_64

$ rpm -qa krb5-server-ldap
krb5-server-ldap-1.10.3-33.el6.x86_64
```

####1.2 openSSL
**如果不配置ssl，这部分可以略过**

OpenLDAP 默认使用 Mozilla NSS，安装后已经生成了一份证书，可使用 certutil -d /etc/openldap/certs/ -L -n 'OpenLDAP Server' 命令查看。使用如下命令生成RFC格式CA证书并分发到客户机待用。

```html
$ certutil -d /etc/openldap/certs/ -L -a -n 'OpenLDAP Server' -f /etc/openldap/certs/password > /etc/openldap/ldapCA.rfc

# 拷贝到其他节点
$ scp /etc/openldap/ldapCA.rfc cdh2:/tmp
$ scp /etc/openldap/ldapCA.rfc cdh3:/tmp
```
附，生成自签名证书的命令供参考：

`$ certutil -d /etc/openldap/certs -S -n 'test cert' -x -t 'u,u,u' -s 'C=XX, ST=Default Province, L=Default City, O=Default Company Ltd, OU=Default Unit, CN=cdh1' -k rsa -v 120 -f /etc/openldap/certs/password`

修改 /etc/sysconfig/ldap，开启 ldaps：
```html
# Run slapd with -h "... ldaps:/// ..."
#   yes/no, default: no
SLAPD_LDAPS=yes
```


####1.3LDAP服务端配置
更新配置库：
```html
rm -rf /var/lib/ldap/*
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
chown -R ldap.ldap /var/lib/ldap
```

在2.4以前的版本中，OpenLDAP 使用 slapd.conf 配置文件来进行服务器的配置，而2.4开始则使用 slapd.d 目录保存细分后的各种配置，这一点需要注意，其数据存储位置即目录 /etc/openldap/slapd.d 。尽管该系统的数据文件是透明格式的，还是建议使用 ldapadd, ldapdelete, ldapmodify 等命令来修改而不是直接编辑。

默认配置文件保存在 /etc/openldap/slapd.d，将其备份：

`cp -rf /etc/openldap/slapd.d /etc/openldap/slapd.d.bak`

添加一些基本配置，并引入 kerberos 和 openldap 的 schema：

```html
$ cp /usr/share/doc/krb5-server-ldap-1.10.3/kerberos.schema /etc/openldap/schema/

$ touch /etc/openldap/slapd.conf

$ echo "include /etc/openldap/schema/corba.schema
include /etc/openldap/schema/core.schema
include /etc/openldap/schema/cosine.schema
include /etc/openldap/schema/duaconf.schema
include /etc/openldap/schema/dyngroup.schema
include /etc/openldap/schema/inetorgperson.schema
include /etc/openldap/schema/java.schema
include /etc/openldap/schema/misc.schema
include /etc/openldap/schema/nis.schema
include /etc/openldap/schema/openldap.schema
include /etc/openldap/schema/ppolicy.schema
include /etc/openldap/schema/collective.schema
include /etc/openldap/schema/kerberos.schema" > /etc/openldap/slapd.conf
$ echo -e "pidfile /var/run/openldap/slapd.pid\nargsfile /var/run/openldap/slapd.args" >> /etc/openldap/slapd.conf

#更新slapd.d
$ slaptest -f /etc/openldap/slapd.conf -F /etc/openldap/slapd.d

$ chown -R ldap:ldap /etc/openldap/slapd.d && chmod -R 700 /etc/openldap/slapd.d
```

####1.4 启动服务

启动 LDAP 服务：
```html
chkconfig --add slapd
chkconfig --level 345 slapd on

/etc/init.d/slapd start
```

查看状态，验证服务端口：
```html
$ ps aux | grep slapd | grep -v grep
  ldap      9225  0.0  0.2 581188 44576 ?        Ssl  15:13   0:00 /usr/sbin/slapd -h ldap:/// -u ldap

$ netstat -tunlp  | grep :389
  tcp        0      0 0.0.0.0:389                 0.0.0.0:*                   LISTEN      8510/slapd
  tcp        0      0 :::389                      :::*                        LISTEN      8510/slapd
```
