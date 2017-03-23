+++
date = "2016-10-19T15:03:24+09:00"
tags = ["centos", "mysql"]
title = "CentOS7にマイナーバージョンのMySQLをyumでインストールしたい"
+++

## はじめに
こんにちは、[えのかわ](https://twitter.com/enkw_)です。  
CentOSにMySQL5.7.11をyumでインストールしたい場面がありました。  
今日（2016/10/19）時点では、最新版は5.7.16です。  
rpmからインストールしようするとpostfixとmariadb-libsのコンフリクトが発生します。  
[CentOS7 で MySQL と postfix を使いたいときの注意点](http://butterfly-effect.hatenadiary.jp/entry/2015/12/19/215901)

## MySQLレポジトリのインストール
まず始めにMySQLレポのインストールです。  
```
$ sudo yum install http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
.
.
=====================================================================================================================================================================================================================
 Package                                                  Arch                                  Version                                 Repository                                                              Size
=====================================================================================================================================================================================================================
Installing:
 mysql57-community-release                                noarch                                el7-7                                   /mysql57-community-release-el7-7.noarch                                7.8 k

Transaction Summary
=====================================================================================================================================================================================================================
Install  1 Package
.
.
$ cat /etc/yum.repos.d/mysql-community.repo
[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://repo.mysql.com/yum/mysql-tools-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Enable to use MySQL 5.5
[mysql55-community]
name=MySQL 5.5 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.5-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Enable to use MySQL 5.6
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

## MySQLのインストール
このまま普通にインストールすると最新のMySQL5.7.16がインストールされてしまいますが、今回はマイナーバージョンを指定してみます。  
この時点でmariadb-libsはeraceされます。
```
$ sudo yum install mysql-community-client-5.7.11-1.el7
=====================================================================================================================================================================================================================
 Package                                                        Arch                                      Version                                         Repository                                            Size
=====================================================================================================================================================================================================================
Installing:
 mysql-community-client                                         x86_64                                    5.7.11-1.el7                                    mysql57-community                                     25 M
 mysql-community-libs                                           x86_64                                    5.7.11-1.el7                                    mysql57-community                                    2.2 M
     replacing  mariadb-libs.x86_64 1:5.5.50-1.el7_2
 mysql-community-libs-compat                                    x86_64                                    5.7.16-1.el7                                    mysql57-community                                    2.0 M
     replacing  mariadb-libs.x86_64 1:5.5.50-1.el7_2
Installing for dependencies:
 mysql-community-common                                         x86_64                                    5.7.11-1.el7                                    mysql57-community                                    270 k

Transaction Summary
=====================================================================================================================================================================================================================
Install  3 Packages (+1 Dependent package)
.
.
Running transaction
  Installing : mysql-community-common-5.7.11-1.el7.x86_64                                                                                                                                                        1/5
  Installing : mysql-community-libs-5.7.11-1.el7.x86_64                                                                                                                                                          2/5
  Installing : mysql-community-client-5.7.11-1.el7.x86_64                                                                                                                                                        3/5
  Installing : mysql-community-libs-compat-5.7.16-1.el7.x86_64                                                                                                                                                   4/5
  Erasing    : 1:mariadb-libs-5.5.50-1.el7_2.x86_64                                                                                                                                                              5/5
  Verifying  : mysql-community-libs-5.7.11-1.el7.x86_64                                                                                                                                                          1/5
  Verifying  : mysql-community-client-5.7.11-1.el7.x86_64                                                                                                                                                        2/5
  Verifying  : mysql-community-common-5.7.11-1.el7.x86_64                                                                                                                                                        3/5
  Verifying  : mysql-community-libs-compat-5.7.16-1.el7.x86_64                                                                                                                                                   4/5
  Verifying  : 1:mariadb-libs-5.5.50-1.el7_2.x86_64                                                                                                                                                              5/5
```

続けてmysql-serverも
```
$ sudo yum install mysql-community-server-5.7.11-1.el7
.
.
=====================================================================================================================================================================================================================
 Package                                                   Arch                                      Version                                              Repository                                            Size
=====================================================================================================================================================================================================================
Installing:
 mysql-community-server                                    x86_64                                    5.7.11-1.el7                                         mysql57-community                                    143 M
Installing for dependencies:
 libaio                                                    x86_64                                    0.3.109-13.el7                                       base                                                  24 k

Transaction Summary
=====================================================================================================================================================================================================================
Install  1 Package (+1 Dependent package)
.
.
```

## MySQLの起動
問題なく起動できるか確認してみます。
```
$ sudo systemctl start mysqld
$ systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2016-10-19 06:08:53 UTC; 18min ago
  Process: 9589 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 9512 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 9593 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─9593 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

Oct 19 06:08:46 ip-10-0-0-205 systemd[1]: Starting MySQL Server...
Oct 19 06:08:53 ip-10-0-0-205 systemd[1]: Started MySQL Server.

$ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.7.11 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## postfix生きてるかな、、、
```
$ rpm -qa | grep postfix
postfix-2.10.1-6.el7.x86_64
```
わぁい。

## さいごに
教えてくださった先輩に感謝です。
