# MySQL部署

## MySQL部署

## DEPLOY\_MYSQL5.7.16\_CentOS7.X.md

描述了系统CentOS7.X下MYSQL5.7.16部署过程和注意事项

### 依赖组件

1. libaio
2. net-tools

### 下载

Mysql安装依赖libaio和net-tools组件，下载地址整理： 1. [mysql安装包](http://192.168.65.191:8082/installs/mysql/mysql-5.7.16-1.el7.x86_64.rpm-bundle.tar) 2. [libaio](http://192.168.65.191:8082/installs/mysql/libaio-0.3.107-10.el6.x86_64.rpm) 3. [net-tools](http://192.168.65.191:8082/installs/mysql/net-tools-2.0-0.22.20131004git.el7.x86_64.rpm)

### 安装步骤

#### 环境检查

CentOS7.X安装默认安装gnome版本时自带了`mariadb`，在安装时，需要检查`mariadb`是否存在，如果存在则需要移除`mariadb`

```bash
[root@dev01 software]# rpm -qa|grep mariadb
 mariadb-libs-5.5.44-2.el7.centos.x86_64 
[root@dev01 software]# rpm -e mariadb-libs-5.5.44-2.el7.centos.x86_64 --nodeps
```

#### 关闭防火墙

这一步骤非常重要，防火墙如果不关闭可能会导致开启`mysql`远程访问时，出现`10060`错误

```bash
[root@localhost software]# systemctl stop firewalld.service 
[root@localhost software]# systemctl disable firewalld.service 
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
Removed symlink /etc/systemd/system/basic.target.wants/firewalld.service.
[root@localhost software]# firewall-cmd --state
not running
[root@localhost software]#
```

#### 安装依赖项

```bash
[root@localhost software]# rpm -Uvh libaio-0.3.107-10.el6.x86_64.rpm
[root@localhost software]# rpm -Uvh net-tools-2.0-0.22.20131004git.el7.x86_64.rpm
```

> CentOS7.X支持软件包指下载不安装，使用--downloadonly --downloaddir=DLDIR两个参数即可实现 \[root@localhost mysql\]\# yum install net-tools --downloadonly --downloaddir=./net-tools

#### 解压mysql安装包

```bash
[root@localhost mysql]# tar -xvf mysql-5.7.16-1.el7.x86_64.rpm-bundle.tar 
mysql-community-libs-compat-5.7.16-1.el7.x86_64.rpm
mysql-community-devel-5.7.16-1.el7.x86_64.rpm
mysql-community-minimal-debuginfo-5.7.16-1.el7.x86_64.rpm
mysql-community-libs-5.7.16-1.el7.x86_64.rpm
mysql-community-common-5.7.16-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.16-1.el7.x86_64.rpm
mysql-community-test-5.7.16-1.el7.x86_64.rpm
mysql-community-embedded-devel-5.7.16-1.el7.x86_64.rpm
mysql-community-server-minimal-5.7.16-1.el7.x86_64.rpm
mysql-community-server-5.7.16-1.el7.x86_64.rpm
mysql-community-client-5.7.16-1.el7.x86_64.rpm
mysql-community-embedded-5.7.16-1.el7.x86_64.rpm
```

#### 安装mysql安装包

```bash
[root@localhost mysql]# rpm -ivh mysql-community-common-5.7.16-1.el7.x86_64.rpm 
[root@localhost mysql]# rpm -ivh mysql-community-libs-5.7.16-1.el7.x86_64.rpm
[root@localhost mysql]# rpm -ivh mysql-community-client-5.7.16-1.el7.x86_64.rpm 
[root@localhost mysql]#  rpm -ivh mysql-community-server-5.7.16-1.el7.x86_64.rpm
[root@localhost mysql]#
```

> 依次执行（几个包有依赖关系，所以执行有先后）

#### 初始化数据库

为了保证数据库目录为与文件的所有者为 mysql 登陆用户，如果你是以 root 身份运行 mysql 服务，需要执行下面的命令初始化

```bash
[root@dev01 software] mysqld --initialize --user=mysql
```

1. 如果是以 mysql 身份运行，则可以去掉 --user 选项。另外 --initialize 选项默认以“安全”模式来初始化，则会为 root 用户生成一个密码并将该密码标记为过期，
2. 登陆后你需要设置一个新的密码，而使用 --initialize-insecure 命令则不使用安全模式，则不会为 root 用户生成一个密码。
3. 这里演示使用的 --initialize 初始化的，会生成一个 root 账户密码，密码在log文件里

   ```bash
   [root@dev01 software]# cat /var/log/mysqld.log 
   2016-12-07T04:41:58.028558Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
   2016-12-07T04:41:58.315188Z 0 [Warning] InnoDB: New log files created, LSN=45790
   2016-12-07T04:41:58.355114Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
   2016-12-07T04:41:58.418448Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 7cc98831-bc37-11e6-a263-000c29f902dc.
   2016-12-07T04:41:58.419935Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
   2016-12-07T04:41:58.420558Z 1 [Note] A temporary password is generated for root@localhost: )1r3gi,hjgQa
   ```

   此时的数据库密码即为：`)1r3gi,hjgQa`

#### 启动数据库

```bash
[root@localhost mysql]# systemctl start mysqld.service
[root@localhost mysql]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.16

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
```

#### 修改用户密码

```bash
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'iflytek';
```

> 以前的 password\(\)函数将会被抛弃，官方建议使用上面的命令来修改密码

#### 调整mysql配置文件

修改my.cnf

```bash
vim /etc/my.cnf
```

在 \[mysqld\] 配置节下追加以下内容

```text
        default-storage-engine=INNODB
        character-set-server=utf8
        collation-server=utf8_general_ci
        lower_case_table_names=1
        max_connections=10000
        sql-mode="ALLOW_INVALID_DATES"
        max_allowed_packet=16M
        bind-address=0.0.0.0
```

> bind-address也可以绑定真实的ip地址 增加mysql配置节

```text
[mysql]
default-character-set=utf8
```

重启mysql服务

```bash
[root@localhost mysql]# systemctl start mysqld.service
```

#### 赋予远程访问

```bash
mysql>  GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'iflytek' WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.01 sec)
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)
```

查看用户设置，确认生效

```bash
mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
Database changed
mysql> select host,user from user;
+-----------+-----------+
| host      | user      |
+-----------+-----------+
| %         | root      |
| localhost | mysql.sys |
| localhost | root      |
+-----------+-----------+
3 rows in set (0.00 sec)
mysql>
```
