+++
date = "2015-07-26T21:06:45+09:00"
tags = ["nagios", "centos", "aws"]
title = "CentOS6.6にNagiosサーバーをインストールする"

+++

## はじめに
こんばんは、インフラエンジニア見習いの[えのかわ](https://twitter.com/enkw_)です。  
Nagiosサーバーの構築をEC2上で行ったので、備忘録として残したいと思います。  
この記事のゴールとしては、Nagiosサーバーから監視対象のサーバーに対してpingでの死活監視ができることとします。


## Nagiosのインストール
NagiosはEPELリポジトリに登録されているので、EPELレポジトリを追加しておきます。
```sh
$ sudo yum install epel-release
$ sudo yum install nagios nagios-plugins-all
```


## アラートメールの設定
Nagiosのアラートメールが送信されるアドレスを登録します。デフォルトの設定では`nagios@localhost`となっているので任意のメールアドレスを設定します。  
念の為にconfigのバックアップをとっておきましょう。
```sh
$ sudo cp -p /etc/nagios/objects/contacts.cfg /etc/nagios/objects/contacts.cfg.sample
$ sudo vi /etc/nagios/objects/contacts.cfg
email                           enokawa@example.com;
```


## Apacheの設定
続いてApacheの設定です。BASIC認証の設定をします。
```sh
$ sudo cp -p /etc/httpd/conf.d/nagios.conf /etc/httpd/conf.d/nagios.conf.sample
$ sudo vi /etc/httpd/conf.d/nagios.conf
ScriptAlias /nagios/cgi-bin/ "/usr/lib64/nagios/cgi-bin/"

<Directory "/usr/lib64/nagios/cgi-bin/">
   Options ExecCGI
   AllowOverride None
   Order allow,deny
   Allow from all
   AuthName "Nagios Access"
   AuthType Basic
   AuthUserFile /etc/nagios/passwd
   Require valid-user
</Directory>

Alias /nagios "/usr/share/nagios/html"

<Directory "/usr/share/nagios/html">
   Options None
   AllowOverride None
   Order allow,deny
   Allow from all
   AuthName "Nagios Access"
   AuthType Basic
   AuthUserFile /etc/nagios/passwd
   Require valid-user
</Directory>
```

続けてベーシック認証のパスワードを設定します。
```sh
$ sudo htpasswd -c /etc/nagios/passwd nagiosadmin
New password:<任意のパスワードを入力>
Re-type new password:<任意のパスワードを再度入力>
```


## 監視ホストの設定
次に監視対象サーバーの設定です。新しく`remotehost.cfg`というファイルを作成しますので、`nagios.cfg`で定義しておきます。

```sh
$ sudo cp -p /etc/nagios/nagios.cfg /etc/nagios/nagios.cfg.sample
$ sudo vi /etc/nagios/nagios.cfg
cfg_file=/etc/nagios/objects/remotehost.cfg   #追加

$ sudo vi /etc/nagios/objects/remotehost.cfg
define host {
    name                        remote-server
    check_interval              10
    retry_interval              1
    max_check_attempts          5
    notification_period         24x7
    contact_groups              admins
    register                    0
}

define host {
    use                         remote-server
    host_name                   <監視対象ホストネーム>
    alias                       i-00000001(なんでもいい)
    address                     <監視対象IPアドレス>
}

define hostgroup {
    hostgroup_name              remote_servers
    alias                       Remote Servers
    members                     <監視対象ホストネーム>
}

define service {
    use                         local-service
    host_name                   <監視対象ホストネーム>
    normal_check_interval       3
    service_description         PING
    check_command               check_ping!100.0,20%!500.0,60%
}
```
※プラグインは`/usr/lib64/nagios/plugins/`配下にあります。


## Security Groupsを開ける
Nagiosサーバーから監視対象サーバーに対してpingを打つので、監視対象サーバーのICMPポートを許可します。  
また、Nagiosサーバーの80番ポートを許可するのも忘れずに行いましょう。


## Apache、Nagiosサーバーのスタート
```sh
$ sudo /etc/init.d/httpd start
Starting httpd:                                            [  OK  ]
$ sudo /etc/init.d/nagios start
Starting nagios: done.
$ sudo chkconfig httpd on
$ sudo chkconfig nagios on
```


## Nagiosの管理画面にアクセスしてみる
以下の情報でアクセスしてみましょう。

* URL：http://NagiosサーバのIP/nagios/
* ユーザ名：nagiosadmin
* パスワード：先ほど設定したパスワード

TOPの`Host Groups`からホスト名`nagios-agent`のリンクをクリックします。
<img src="/images/nagios-server1.png">

このように表示されていたらOKです。
<img src="/images/nagios-server2.png">


## アラートが飛ぶことを確認する
試しにSecutiry GroupsのICMPポートを閉じてみて、アラートメールが送信されることを確認してみましょう。  
Nagiosの管理画面ではこのようなアラートがあがります。
<img src="/images/nagios-server3.png">

ちゃんとメールも飛んできてますね。
<img src="/images/nagios-server4.png">

復旧すると復旧した旨のメールが届きます。


## おわりに
次回は監視対象サーバーのログ監視を行いたいと思います。


## 参考
* CentOS 6.4にサーバ統合監視ツールのNagiosを使ってWEBサイトの死活監視する http://j-caw.co.jp/blog/?p=1125
