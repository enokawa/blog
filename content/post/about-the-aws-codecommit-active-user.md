+++
date = "2016-10-20T13:05:13+09:00"
tags = ["aws", "codecommit"]
title = "AWS CodeCommitのアクティブユーザについて"
+++

## はじめに
こんにちは、[えのかわ](https://twitter.com/enkw_)です。  
CodeCommitのアクティブユーザの定義について意味を理解していなかったので懺悔します。

## 甘かった僕の認識
`最初の 5 人のアクティブユーザー*は無料」`という表示で、「あっ、5つのIAMユーザまでは無料なんだ」と勘違いしていました。  しかし実際のドキュメントの内容は下記で、完璧に意味を履き違えていました。

>* アクティブユーザーとは、その月に Git リクエストまたは AWS マネジメントコンソールを使用して AWS CodeCommit リポジトリにアクセスする、すべての固有の AWS Identity (IAM ユーザー、IAM ロール、フェデレーティッドユーザー、ルートアカウント) を指します。一意の AWS アイデンティティを使用して CodeCommit にアクセスするサーバーは、アクティブなユーザーとみなされます。その月に AWS CodeCommit にアクセスしていないユーザーに対しては料金が発生しません。ストレージには、リポジトリのデータを保持するために必要な容量全体が含まれます。

料金 - Amazon CodeCommit | AWS  https://aws.amazon.com/jp/codecommit/pricing/
## ということは
例えば5つのクライアントPCで同じIAMユーザ(SSH Key ID)を利用(Pull or Push)した場合、5アクティブユーザとしてみなされます。  僕の場合、1台のクライアントPCでPushやPullを行い、6台のEC2でpullのみを行いました。その結果`additional CodeCommit user: 2 User-Month`としてみなされました。

## さいごに
AWSのドキュメントはちゃんと読もうな、オレ。
