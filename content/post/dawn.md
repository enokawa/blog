+++
date = "2018-02-14T00:26:12+09:00"
tags = ["aws", "ec2", "serverless", "python"]
title = "Serverless FrameworkでEC2を定期的に起動/停止するdawnを公開しました"
+++

## はじめに
こんにちは[えのかわ](https://twitter.com/enkw_)です。タイトルの通り、EC2 を定期的に起動/停止するツール、名付けて dawn をリリースしました。  
[enokawa/dawn: Lambda function to automaticaly stop and start the EC2 instance.](https://github.com/enokawa/dawn)

## dawn??
dawn とは **夜明け、明け方** という意味です。名付け親は [id:iga-ninja](http://iga-ninja.hatenablog.com/about) さんです。どうも良い名前が決まらなくて決めてもらいました。発音は **dän, dôn** だそうです。どぁ〜ん。

## きっかけ
業務のなかで、よく EC2 を定期的に停止/起動するケースがあるので、どうせなら [Serverless Framework](https://serverless.com/) でと思い作成しました。けっこうありきたりですが、EC2 の費用削減にもなりますし僕も利用者の一人です。

また、dawn は EC2 の AMI をスケジュールで取得する [y13i/amirotate](https://github.com/y13i/amirotate) を参考にして作成しました。amirotate に限らず、y13i さんのレポジトリはすごく参考になります。

## おわりに
今回は単純なスクリプトですが、今後もガンガン Serverless Framework 使っていきます。何か要望などあれば Pull Request いただけると僕が喜びます。
