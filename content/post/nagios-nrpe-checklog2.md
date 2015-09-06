+++
date = "2015-07-27T21:06:43+09:00"
tags = ["nagios", "centos", "aws"]
title = "NagiosのNRPEプラグインを使ってログ監視をする"

+++

## はじめに
こんばんは、インフラエンジニア見習いの[えのかわ](https://twitter.com/enkw_)です。  
[前回の記事](/2015/07/26/nagios-server/)の続きです。今回は監視対象のログ監視をしたいと思います。


## NRPE is 何
NRPE(Nagios Remote Plugin Executor)とは、監視対象サーバーに対して以下のような監視を実現したい時に用います。前回の記事ではpingを打つだけなのでNRPEは必要ではありませんでした。

* ディスク監視
* CPU使用率の監視
* メモリ使用率の監視
* ログ監視


## NRPEのインストール（Nagiosサーバー）
それではさっそく設定していきます。まずはじめにNagiosサーバーにNRPE関連のパッケージをインストールします。
`/usr/lib64/nagios/plugins/`配下に`check_nrpe`というファイルがインストールされます。
```sh
$ sudo yum install nrpe nagios-plugins-nrpe
```

インストールした`check_nrpe`コマンドを登録します。
```sh
$ sudo vi /etc/nagios/objects/commands.cfg
define command{
        command_name    check_nrpe
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
      }
```


## NRPEのインストールと設定（監視対象サーバ）
つづいて監視対象サーバに、NRPE関連のパッケージをインストールします。
```sh
$ sudo yum install -y epel-release
$ sudo yum install nrpe nagios-plugins-nrpe nagios-plugins-all
```

`nrpe.cfg`に設定を書いていきます。Nagiosサーバーからのアクセスを許可します。
```sh
$ sudo cp -p /etc/nagios/nrpe.cfg.sample
$ sudo vi /etc/nagios/nrpe.cfg
allowed_hosts=127.0.0.1, <Nagiosサーバーのip>

$ sudo  chkconfig nrpe on
$ sudo /etc/init.d/nrpe start
```
Nagiosサーバーから監視対象サーバーに対してNRPEコマンドを実施する際に、5666番ポートで通信しますので監視対象サーバーの5666番ポートを許可します。  


## `check_log2`のインストールと設定（監視対象サーバ）
今回は`check_log2`というスクリプトを使います。`check_log3`というのもあって、`check_log3`は正規表現を用いたログ監視も行えるそうです。
`/var/log/messages`の中に`ERR`という文字列が含まれていたら、Criticalなアラートが上がるように設定してみます。権限にも注意しましょう。
```sh
$ cd /usr/lib64/nagios/plugins/
$ sudo curl -L https://raw.githubusercontent.com/dnsmichi/nagiosplugins/master/contrib/check_log2.pl > check_log2
$ sudo chmod 755 check_log2
$ cd

$ sudo touch /tmp/.messages_old
$ sudo chmod 766 /tmp/.messages_old
$ sudo chmod 644 /var/log/messages
$ sudo vi /etc/nagios/nrpe.cfg
command[check_log2]=/usr/lib64/nagios/plugins/check_log2 -l /var/log/messages -s /tmp/.messages_old -p ERR -c
```


## nagiosユーザーに権限を与える
NRPEコマンドがNagiosサーバーで実施される時は、実際はnagiosユーザーが`check_log2`などのコマンドを実行します。`visudo`でnagiosユーザーが`check_log2`コマンドを実施できるように設定します。
```sh
$ sudo visudo
nagios  ALL=NOPASSWD:/usr/lib64/nagios/plugins/check_log2
```


## 実際にコマンドを打ってテストしてみる（Nagiosサーバー）
Nagiosサーバーからコマンドを打ってみます。
```sh
[nagios-server]$ cd /usr/lib64/nagios/plugins/
[nagios-server]$ ./check_nrpe -H <監視対象サーバーのIP> -c check_log2
OK - No matches found.
```

エラーが出るかも試してみます。`/var/log/messages`に`logger`コマンドでエラーログを吐いてみます。
```sh
[nagios-agent]$ logger "ERR"
```

ちゃんとエラーが出ていますね。
```sh
[nagios-server]$ ./check_nrpe -H <監視対象サーバーのIP> -c check_log2
CRITICAL: (1): Jul 27 21:44:16 ip-XX-XX-XX-XX ec2-user: ERR
```


## `remotehost`設定（Nagiosサーバー）
前回と同じ要領で`remotehost.cfg`に追記していきます。
```sh
$ sudo vi /etc/nagios/objects/remotehost.cfg
define service{
    use                         generic-service
    host_name                   nagios-agent
    normal_check_interval       3
    service_description         Log Check
    check_command               check_nrpe!check_log2
}

$ sudo /etc/init.d/nagios restart
```

監視できてますね！
<img src="/images/nagios-nrpe-checklog2-1.png">

アラートもあがります！
<img src="/images/nagios-nrpe-checklog2-2.png">


## おわりに
次回はAWSのCloudWatchのログを監視してみたいと思います！


## 参考
* Welcome to Lonnie's site. - NRPEを使い別ホストのlogをNagiosで監視する http://www.lonnie.co.jp/xoops/modules/d3forum/index.php?topic_id=22
* Nagiosでログ監視を行う http://blog.cloudpack.jp/2011/12/01/server-news-nagios-log-monitoring/
* NRPEのログを出力する方法(英語) http://www.charlesjudith.com/2014/03/11/create-a-log-file-for-nrpe/
* nagiosってなんじゃ？（nrpe + check_log3 + negateでログに特定のキーワードが含まれていなかったらアラート） http://memocra.blogspot.com/2013/05/nagiosnrpe-checklog3-negate.html?spref=tw
* NRPEを使った監視を実装する - think-t の晴耕雨読 http://think-t.hatenablog.com/entry/20111231/p1
* 他のサーバーのリソース監視する方法 http://good-stream.com/goodstream/nagios/nrpe.html
* nagiosにcheck_nrpe導入後のトラブルシュート - fs随筆
http://d.hatena.ne.jp/foursue/20090717/1247796901
