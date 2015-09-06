+++
date = "2015-08-09T16:22:36+09:00"
tags = ["skyhopper", "aws"]
title = "DevOps採用システム自動構築ツール「SkyHopper」を使ってみた"

+++

## はじめに
こんにちは、インフラエンジニア見習いの[えのかわ](https://twitter.com/enkw_)です。  
今回は[株式会社スカイアーチネットワークス](http://www.skyarch.net/)が開発したDevOps採用システム自動構築ツール「[SkyHopper](http://www.skyarch.net/blog/?p=2709)」を使ってみましたのでそのレポートを書きたいと思います。

今回はAmazon Linuxのt2.microを用いて構築します。
ゴールとしては、SkyHopperを利用してELB+EC2+RDSのよく見る構成を構築します。
<img src="/images/skyhopper.png">

## SkyHopperとは
ひと言でいうと、「ブラウザでAWSの構築ができてchefやServerspecが流せて監視ができるもの」ですかね。長いですね。笑


## SkyHopperのインストール
基本的には[デプロイ手順](https://github.com/skyarch-networks/skyhopper/blob/master/doc/installation/skyhopper.md)を順番に進めていけば構築できます。
[Cookbook](https://github.com/skyarch-networks/skyhopper_cookbooks/tree/master/cookbooks/skyhopper)も用意されているのでChefに慣れている方はChefで構築した方が楽かもしれません。


## セットアップ
以下の図のように設定します。
<img src="/images/skyhopper1.png">


構築が完了したら以下のコマンドを打ってskyhopperを再起動します。
```sh
$ cp -r ~/skyhopper/tmp/chef ~/.chef
$ ./scripts/skyhopper_daemon.sh stop
$ ./scripts/skyhopper_daemon.sh start
```

自動的に2台のインスタンスが構築されます。おそらくChefサーバーとZabbixサーバーです。EIPも関連付けられます。
<img src="/images/skyhopper2.png">


## サインアップ
サインアップします。`master`と`admin`の意味はまだ分かってないです。分かり次第、更新したいと思います。
<img src="/images/skyhopper3.png">


## 顧客の作成
えのかわ株式会社から受注しました。顧客コードの決め方は規約を設けた方がいいと思いました。
<img src="/images/skyhopper4.png">


## 案件の作成
えのかわ株式会社から公式HP作成の依頼が来たので新しく案件を作成します。アクセスキーとシークレットアクセスキーはお客様から頂きました。
<img src="/images/skyhopper5.png">


## 新規インフラの作成
新しくインフラを構築します。EC2インスタンスも構築するので新しくキーペアーも登録します。新しく作成することもできます。
<img src="/images/skyhopper6.png">


## スタックの詳細（1）
あらかじめ用意されているCloudFormationのテンプレートを用います。僕はWordPressのAMIを作成して、テンプレートにAMIを指定しました。
<img src="/images/skyhopper7.png">

## スタックの詳細（2）
インスタンスタイプやDBの設定をすることができます。「送信」ボタンを押すと、CloudFormationのスタックが実行されます。
<img src="/images/skyhopper8.png">


## ブラウザでアクセスしてみる
ELBのDNSでアクセスしてみます。お決まりのWordPressのセットアップ画面が表示されます。「データベースホスト」欄にRDSのDNS Nameを入力すれば完了です。2台のEC2のアクセスログをtailして負荷分散されることを確認します。
<img src="/images/skyhopper9.png">

## ハマったところ
* 鍵ペア名に`.pem`をいれて`stack creation failed`エラー
* Default VPCが存在していなくてエラー


## 気になったところ
* ログにACCESSKEYとかSECRETACCESSKEYとか秘密鍵を吐いてるので気をつける必要があると思いました。  
