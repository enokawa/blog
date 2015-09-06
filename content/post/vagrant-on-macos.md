+++
date = "2015-09-06T16:57:47+09:00"
tags = ["vagrant", "mac"]
title = "VagrantでWordPress環境を作る（MacOSX編）"

+++

## はじめに
こんにちは、[えのかわ](https://twitter/enkw_)です。  
今回の記事は自分用の備忘録です。WordPress環境の作成を目標に進めていきます。  
構成は以下のような感じです。

* CentOS 6.5
* Apache 2.2系
* MySQL 5.1系
* PHP 5.3系
* WordPress 最新版


## VirtualBoxのインストール
[VirtualBoxのダウンロードページ(Old Builds)](https://www.virtualbox.org/wiki/Download_Old_Builds_5_0)からVirtual5.0をインストールします。最新バージョン(5.02)だと[vagrant upでコケる](http://qiita.com/medaka5/items/d70751a562a5604c2115)みたいです。
環境にもよりますが、インストールに5分ほど時間かかります。 `VirtualBox.pkj`をダブルクリックしてウィザードに従って進めてください。
<img src="/images/vagrant-on-macos1.png">

## Vagrantのインストール
[Vagrantのダウンロードページ](https://www.vagrantup.com/downloads.html)からインストールします。
こちらもインストールに5分ほどかかります。VirtualBoxと同様に、ウィザードに従って進めていきます。
<img src="/images/vagrant-on-macos2.png">

## Boxのインストール
OSのイメージであるBoxをインストールしてVagrantのセットアップを行います。まず適当なディレクトリを作成します。
```sh
$ mkdir mkdir vagrant
$ mkdir vagrant/CentOS65
$ cd vagrant/CentOS65
```

次にCentOS6.5のBoxをインストールします。  
Boxの一覧は、http://www.vagrantbox.es/ にあります。こちらも結構時間かかります。
```sh
$ vagrant box add centos65 https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos65' (v0) for provider:
    box: Downloading: https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box
==> box: Successfully added box 'centos65' (v0) for 'virtualbox' !
```


## OSの起動
CentOS6.5を起動します。`vagrant ssh`で仮想マシンにsshログインができます。`vagrant init`でVagrantの設定ファイルとなる`Vagrantfile`が生成されます。
```sh
$ vagrant init centos65
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 => 2222 (adapter 1)
    default: /vagrant => /Users/enokawa/vagrant/CentOS65
    .
    .
    .
$ vagrant ssh
[vagrant@vagrant-centos65 ~]$
```


## LAMPのインストール/起動
Apache、MySQL、PHPをインストールします。バージョンや設定などは省略しています。
```sh
[vagrant@vagrant-centos65 ~]$ sudo yum update -y
[vagrant@vagrant-centos65 ~]$ sudo reboot
[vagrant@vagrant-centos65 ~]$ sudo yum -y install httpd httpd-devel
[vagrant@vagrant-centos65 ~]$ sudo yum -y install php php-mysql php-xml php-pear php-pdo php-cli php-mbstring php-gd php-mcrypt php-common php-devel php-bcmath
[vagrant@vagrant-centos65 ~]$ sudo yum -y install mysql mysql-devel mysql-server
[vagrant@vagrant-centos65 ~]$ sudo /etc/init.d/httpd start
[vagrant@vagrant-centos65 ~]$ sudo chkconfig httpd on
[vagrant@vagrant-centos65 ~]$ sudo /etc/init.d/mysqld start
[vagrant@vagrant-centos65 ~]$ sudo chkconfig mysqld on
```

## mysql-serverの設定
WordPress用にデータベースの設定します。新しく`wordpress`ユーザーを作成します。
```sh
[vagrant@vagrant-centos65 ~]$ mysql -u root -p
Enter password:<空Enter>
mysql> CREATE DATABASE wordpress;
mysql> grant all privileges on wordpress.* to wordpress@localhost identified by '任意のパスワード';
mysql> flush privileges;
mysql> quit;
```

## WordPressのインストール
Mac上でWordPressを[公式ページ](https://ja.wordpress.org/)からインストールして解凍します。今回はデスクトップ上に展開したいと思います。
<img src="/images/vagrant-on-macos3.png">

<img src="/images/vagrant-on-macos4.png">


## クライアントPCとディレクトリの同期
`Vagrantfile`を編集します。  
以下の2行の下に
```sh
# config.vm.network "private_network", ip: "192.168.33.10"
# config.vm.synced_folder "../data", "/vagrant_data"
```
以下の2行を追加します。
```sh
config.vm.network "private_network", ip: "192.168.33.10"
config.vm.synced_folder "~/Desktop/wordpress", "/var/www/html", owner: "apache", group: "apache"
```
その後に`vagrant reload`で設定ファイルを再読み込みします。
```sh
$ vagrant reload
```

## ブラウザでアクセスしてみる
Mac上のブラウザで http://192.168.33.10 にアクセスします。  
おなじみの画面がでてきますので説明に従って進めていきます。
<img src="/images/vagrant-on-macos5.png">

<img src="/images/vagrant-on-macos6.png">

<img src="/images/vagrant-on-macos7.png">

<img src="/images/vagrant-on-macos8.png">
記事を投稿できました！


## 参考サイト
* Vagrant日本語ドキュメント - http://blog.raqda.com/vagrant/index.html
