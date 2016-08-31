
=====================
HDFS配置Kerberos认证
======================


###1. 环境说明

**系统环境：**
```html
操作系统：CentOs 6.6
Hadoop版本：CDH5.4
JDK版本：1.7.0_71
运行用户：root
```

**集群各节点角色规划为：**

```html
192.168.56.121        cdh1     NameNode、ResourceManager、HBase、Hive metastore、Impala Catalog、Impala statestore、Sentry 
192.168.56.122        cdh2     DataNode、SecondaryNameNode、NodeManager、HBase、Hive Server2、Impala Server
192.168.56.123        cdh3     DataNode、HBase、NodeManager、Hive Server2、Impala Server
```
cdh1作为master节点，其他节点作为slave节点，我们在cdh1节点安装kerberos Server，在其他节点安装kerberos client。



###2.准备工作

确认添加主机名解析到 /etc/hosts 文件中。

```html
$ cat /etc/hosts
127.0.0.1       localhost

192.168.56.121 cdh1
192.168.56.122 cdh2
192.168.56.123 cdh3

注意：hostname 请使用小写，要不然在集成 kerberos 时会出现一些错误。
```

### 3.安装kerberos
在 cdh1 上安装包 krb5、krb5-server 和 krb5-client。

`$ yum install krb5-server -y`

在其他节点（cdh1、cdh2、cdh3）安装 krb5-devel、krb5-workstation ：

```html
#使用无密码登陆
$ ssh cdh1 "yum install krb5-devel krb5-workstation -y"
$ ssh cdh2 "yum install krb5-devel krb5-workstation -y"
$ ssh cdh3 "yum install krb5-devel krb5-workstation -y"
```

###4.修改配置文件

kdc 服务器涉及到三个配置文件：
```html
/etc/krb5.conf
/var/kerberos/krb5kdc/kdc.conf
/var/kerberos/krb5kdc/kadm5.acl
```

配置 Kerberos 的一种方法是编辑配置文件 /etc/krb5.conf。默认安装的文件中包含多个示例项。

```html
[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 default_realm = JAVACHEN.COM
 dns_lookup_realm = false
 dns_lookup_kdc = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 default_tgs_enctypes = aes256-cts-hmac-sha1-96
 default_tkt_enctypes = aes256-cts-hmac-sha1-96
 permitted_enctypes = aes256-cts-hmac-sha1-96
 clockskew = 120
 udp_preference_limit = 1

[realms]
 JAVACHEN.COM = {
  kdc = cdh1
  admin_server = cdh1
 }

[domain_realm]
 .javachen.com = JAVACHEN.COM
 javachen.com = JAVACHEN.COM
```

说明：
```html
[logging]：表示 server 端的日志的打印位置

[libdefaults]：每种连接的默认配置，需要注意以下几个关键的小配置

default_realm = JAVACHEN.COM：设置 Kerberos 应用程序的默认领域。如果您有多个领域，只需向 [realms] 节添加其他的语句。

ticket_lifetime： 表明凭证生效的时限，一般为24小时。

renew_lifetime： 表明凭证最长可以被延期的时限，一般为一个礼拜。当凭证过期之后，对安全认证的服务的后续访问则会失败。

clockskew：时钟偏差是不完全符合主机系统时钟的票据时戳的容差，超过此容差将不接受此票据。通常，将时钟扭斜设置为 300 秒（5 分钟）。这意味着从服务器的角度看，票证的时间戳与它的偏差可以是在前后 5 分钟内。

udp_preference_limit= 1：禁止使用 udp 可以防止一个 Hadoop 中的错误

[realms]：列举使用的 realm。

kdc：代表要 kdc 的位置。格式是 机器:端口

admin_server：代表 admin 的位置。格式是 机器:端口

default_domain：代表默认的域名

[appdefaults]：可以设定一些针对特定应用的配置，覆盖默认配置。
```

修改 /var/kerberos/krb5kdc/kdc.conf ，该文件包含 Kerberos 的配置信息。例如，KDC 的位置，Kerbero 的 admin 的realms 等。需要所有使用的 Kerberos 的机器上的配置文件都同步。这里仅列举需要的基本配置。

```html
[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 JAVACHEN.COM = {
  #master_key_type = aes256-cts
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  max_renewable_life = 7d
  max_life = 1d
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
  default_principal_flags = +renewable, +forwardable
 }
```

说明：
```html
JAVACHEN.COM： 是设定的 realms。名字随意。Kerberos 可以支持多个 realms，会增加复杂度。大小写敏感，一般为了识别使用全部大写。这个 realms 跟机器的 host 没有大关系。

master_key_type：和 supported_enctypes 默认使用 aes256-cts。JAVA 使用 aes256-cts 验证方式需要安装 JCE 包，见下面的说明。为了简便，你可以不使用 aes256-cts 算法，这样就不需要安装 JCE 。

acl_file：标注了 admin 的用户权限，需要用户自己创建。文件格式是：Kerberos_principal permissions [target_principal] [restrictions]
su
pported_enctypes：支持的校验方式。

admin_keytab：KDC 进行校验的 keytab。
```

为了能够不直接访问 KDC 控制台而从 Kerberos 数据库添加和删除主体，请对 Kerberos 管理服务器指示允许哪些主体执行哪些操作。通过编辑文件 /var/lib/kerberos/krb5kdc/kadm5.acl 完成此操作。ACL（访问控制列表）允许您精确指定特权。

```html
$ cat /var/kerberos/krb5kdc/kadm5.acl
  */admin@JAVACHEN.COM *
```


###5.同步配置文件

将 kdc 中的 /etc/krb5.conf 拷贝到集群中其他服务器即可。

```html
$ scp /etc/krb5.conf cdh2:/etc/krb5.conf
$ scp /etc/krb5.conf cdh3:/etc/krb5.conf
```
请确认集群如果关闭了 selinux。


###6.创建数据库
在 cdh1 上运行初始化数据库命令。其中 -r 指定对应 realm。

`$ kdb5_util create -r JAVACHEN.COM -s`

出现 Loading random data 的时候另开个终端执行点消耗CPU的命令如 cat /dev/sda > /dev/urandom 可以加快随机数采集。该命令会在 /var/kerberos/krb5kdc/ 目录下创建 principal 数据库。

如果遇到数据库已经存在的提示，可以把 /var/kerberos/krb5kdc/ 目录下的 principal 的相关文件都删除掉。默认的数据库名字都是 principal。可以使用 -d 指定数据库名字。

###7. 启动服务
在 cdh1 节点上运行：

```html
$ chkconfig --level 35 krb5kdc on
$ chkconfig --level 35 kadmin on
$ service krb5kdc start
$ service kadmin start
```

###8.创建kerberos管理员
关于 kerberos 的管理，可以使用 kadmin.local 或 kadmin，至于使用哪个，取决于账户和访问权限：

如果有访问 kdc 服务器的 root 权限，但是没有 kerberos admin 账户，使用 kadmin.local
如果没有访问 kdc 服务器的 root 权限，但是用 kerberos admin 账户，使用 kadmin

在 cdh1 上创建远程管理的管理员：
```html
#手动输入两次密码，这里密码为 root
$ kadmin.local -q "addprinc root/admin"

# 也可以不用手动输入密码
$ echo -e "root\nroot" | kadmin.local -q "addprinc root/admin"

# 或者运行下面命令
$ kadmin.local <<eoj
addprinc -pw root root/admin
eoj
```
系统会提示输入密码，密码不能为空，且需妥善保存。


###9.测试kerberos
查看当前的认证用户：

```html
# 查看principals
$ kadmin: list_principals

  # 添加一个新的 principal
  kadmin:  addprinc user1
    WARNING: no policy specified for user1@JAVACHEN.COM; defaulting to no policy
    Enter password for principal "user1@JAVACHEN.COM":
    Re-enter password for principal "user1@JAVACHEN.COM":
    Principal "user1@JAVACHEN.COM" created.

  # 删除 principal
  kadmin:  delprinc user1
    Are you sure you want to delete the principal "user1@JAVACHEN.COM"? (yes/no): yes
    Principal "user1@JAVACHEN.COM" deleted.
    Make sure that you have removed this principal from all ACLs before reusing.

  kadmin: exit
```

也可以直接通过下面的命令来执行：

```html
# 提示需要输入密码
$ kadmin -p root/admin -q "list_principals"
$ kadmin -p root/admin -q "addprinc user2"
$ kadmin -p root/admin -q "delprinc user2"

# 不用输入密码
$ kadmin.local -q "list_principals"
$ kadmin.local -q "addprinc user2"
$ kadmin.local -q "delprinc user2"
```
创建一个测试用户 test，密码设置为 test：

`$ echo -e "test\ntest" | kadmin.local -q "addprinc test"`

获取 test 用户的 ticket：

```html
# 通过用户名和密码进行登录
$ kinit test
Password for test@JAVACHEN.COM:

$ klist  -e
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: test@JAVACHEN.COM

Valid starting     Expires            Service principal
11/07/14 15:29:02  11/08/14 15:29:02  krbtgt/JAVACHEN.COM@JAVACHEN.COM
  renew until 11/17/14 15:29:02, Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96

Kerberos 4 ticket cache: /tmp/tkt0
klist: You have no tickets cached
```

销毁该 test 用户的 ticket：

```html
$ kdestroy

$ klist
klist: No credentials cache found (ticket cache FILE:/tmp/krb5cc_0)

Kerberos 4 ticket cache: /tmp/tkt0
klist: You have no tickets cached
```

更新ticket：
```html
$ kinit root/admin
  Password for root/admin@JAVACHEN.COM:

$  klist
  Ticket cache: FILE:/tmp/krb5cc_0
  Default principal: root/admin@JAVACHEN.COM

  Valid starting     Expires            Service principal
  11/07/14 15:33:57  11/08/14 15:33:57  krbtgt/JAVACHEN.COM@JAVACHEN.COM
    renew until 11/17/14 15:33:57

  Kerberos 4 ticket cache: /tmp/tkt0
  klist: You have no tickets cached

$ kinit -R

$ klist
  Ticket cache: FILE:/tmp/krb5cc_0
  Default principal: root/admin@JAVACHEN.COM

  Valid starting     Expires            Service principal
  11/07/14 15:34:05  11/08/14 15:34:05  krbtgt/JAVACHEN.COM@JAVACHEN.COM
    renew until 11/17/14 15:33:57

  Kerberos 4 ticket cache: /tmp/tkt0
  klist: You have no tickets cached
```

抽取密钥并将其储存在本地 keytab 文件 /etc/krb5.keytab 中。这个文件由超级用户拥有，所以您必须是 root 用户才能在 kadmin shell 中执行以下命令：

```html
$ kadmin.local -q "ktadd kadmin/admin"

$ klist -k /etc/krb5.keytab
  Keytab name: FILE:/etc/krb5.keytab
  KVNO Principal
  ---- --------------------------------------------------------------------------
     3 kadmin/admin@LASHOU-INC.COM
     3 kadmin/admin@LASHOU-INC.COM
     3 kadmin/admin@LASHOU-INC.COM
     3 kadmin/admin@LASHOU-INC.COM
     3 kadmin/admin@LASHOU-INC.COM
```





