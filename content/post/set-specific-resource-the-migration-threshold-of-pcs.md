+++
date = "2016-10-27T17:59:37+09:00"
tags = ["pcs"]
title = "【pcs】特定リソースのmigration-thresholdを変更する"
+++

## はじめに
こんにちは、[えのかわ](https://twitter.com/enkw_)です。  
Pacemaker/Corosyncで、特定リソースのmigration-thresholdを変更してみたのでメモです。  
基本的にはRed Hatのドキュメントを読めば理解できると思います。  
[Red Hat Enterprise Linux 6 Pacemaker を使用した Red Hat High Availability Add-On の設定](https://access.redhat.com/documentation/ja-JP/Red_Hat_Enterprise_Linux/6/html/Configuring_the_Red_Hat_High_Availability_Add-On_with_Pacemaker/)

pcsのバージョンは下記。AmazonLinuxです。
```
$ pcs --version
0.9.141
```

## ざっくり
全体のmigration-thresholdは1で設定しています。  
今回はapacheリソースのみ2に変更してみます。
```
$ sudo pcs resource defaults
resource-stickiness: INFINITY
migration-threshold: 1
```
migration-thresholdの変更
```
$ sudo pcs resource meta apache migration-threshold=2
```
migration-thresholdの確認
```
$ sudo pcs resource show apache
 Resource: apache (class=lsb type=apache)
  Meta Attrs: migration-threshold=2
  Operations: start on-fail=restart interval=0s timeout=20s (apache-start-interval-0s)
              monitor on-fail=restart interval=60s timeout=60s (apache-monitor-interval-60s)
              stop on-fail=fence interval=0s timeout=20s (apache-stop-interval-0s)
```
migration-thresholdを2以上に変更した場合、failure-timeoutの変更も併せて必要になります。
```
$ sudo pcs resource meta apache failure-timeout=180s
```
failure-timeoutの確認
```
$ sudo pcs resource show apache
 Resource: apache (class=lsb type=apache)
  Meta Attrs: migration-threshold=2 failure-timeout=180s
  Operations: start on-fail=restart interval=0s timeout=20s (apache-start-interval-0s)
              monitor on-fail=restart interval=60s timeout=60s (apache-monitor-interval-60s)
              stop on-fail=fence interval=0s timeout=20s (apache-stop-interval-0s)
```

## メモ
- fail-countが1加算されるとリソース(monitor)の再起動が発生する
- fail-countがmigration-thresholdを超過するとfail-countはINFINITYとなり即時FOが発生する
- 再起動を試みて、問題なくリソースが動作すればFOは発生しない
- fail-countはfailure-timeout値を過ぎるとresetされる(ログにも残る)
- 設定するのは片系のみで問題ない
- Pacemaker/Corosyncの再起動は不要

## さいごに
今回は下記のページを参考にしました。  
[Pacemaker を使用した Red Hat High Availability Add-On の設定 5.4. リソースのメタオプション](https://access.redhat.com/documentation/ja-JP/Red_Hat_Enterprise_Linux/6/html/Configuring_the_Red_Hat_High_Availability_Add-On_with_Pacemaker/s1-resourceopts-HAAR.html)  
[Pacemaker を使用した Red Hat High Availability Add-On の設定 7.2. 障害発生のためリソースを移動する](https://access.redhat.com/documentation/ja-JP/Red_Hat_Enterprise_Linux/7/html/High_Availability_Add-On_Reference/s1-failure_migration-HAAR.html)  
[Qiita - AWS/EC2 Corosync PacemakerでNFSを冗長化する](http://qiita.com/SatoHiroyuki/items/ef52d17ddb3bf9afcb6a)
