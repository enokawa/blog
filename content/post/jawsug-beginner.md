+++
date = "2015-07-25T22:57:26+09:00"
tags = ["jawsug", "aws"]
title = "JAWS-UG初心者支部【第2回】懇親会でLTしてきました！！ #jawsug_bgnr"

+++

# はじめに
みなさんこんにちは、インフラエンジニア見習いの[えのかわ](https://twitter.com/enkw_)です。  
先週の金曜日に行われたJAWS-UG初心者支部の懇親会でLTをしてきたのでそのレポートを書きます！

イベントページはこちら
https://jawsug-beginner.doorkeeper.jp/events/26430  
togetterはこちら
http://togetter.com/li/848886

運営の加我さん参加者のレポートです。加我さんからは写真をお借りしてます！ありがとうございます！  
第2回JAWS-UG 初心者支部(7/17) - #ダメなら餃子 http://damenaragyouza.hatenablog.jp/entry/2015/07/21/143643

## 経緯
cloudpackエバンジェリストの[@yoshidashingo](https://twitter.com/yoshidashingo)さんに「LTしておいでや」と
言われたので久々に勉強会に参加させていただきました。正直、時間的にLTできないと思っていたので、ラッキーでした。  
LTさせてくれた山崎さん、青木さん、運営のみなさん、ありがとうございました！！

<img src="/images/jawsug-beginner.jpg">
## 勉強会
正直、勉強会自体は資料作りに夢中であまり話を聞けていたなかったので[加我さんのブログ](http://damenaragyouza.hatenablog.jp/entry/2015/07/21/143643)を見ると雰囲気けっこう掴めると思います。次からは事前に準備して臨みたいと思います！


## Piculet
今回のLTのテーマは「AWS初心者がCodenize.toolsでInfrastructure as Codeした話」です。
[Codenize.tools](http://codenize.tools/)とはAWSのサービスをマイグレーション(移行)するツール群です。クックパッドの[@sgwr_dts](https://twitter.com/sgwr_dts)さんが作成しています。
今回は、その中でもAWSのSecurity Groupsをマイグレーションするツールである[Piculet](http://piculet.codenize.tools/)について紹介しました！

Piculetの良さは`--dry-run`オプションが使える点です。applyする前に`--dry-run`で流して、他の人にレビューしてもらった後に
applyする流れがいいと思います。Github(プライベートリポジトリなど)でレビューをしてもらうのもアリだと思います。
```bash
$ piculet -a -p prod -r ap-northeast-1 --dry-run
```


<script async class="speakerdeck-embed" data-id="d03f788e42ae49049a0f54d0e000a406" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

## IAMユーザーの権限
Piculetを使用するIAMユーザーのポリシーは以下のような感じです。SGだけを操作できる最低限の権限だけを渡しておきます。
IP制限をかけておくとよりセキュアに運用できると思います。
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt0000000000001",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeTags",
                "ec2:AuthorizeSecurityGroupEgress",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:RevokeSecurityGroupEgress",
                "ec2:CreateSecurityGroup",
                "ec2:DeleteSecurityGroup"
            ],
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "XX.XX.XX.XX"
                }
            },
            "Resource": [
                "*"
            ]
        }
    ]
}
```


## おわりに
今回はPiculetを紹介しましたが、Codenize.toolsでは、他にもRoute53のレコードをマイグレーションする[Roadworker](http://roadworker.codenize.tools/)やELBをマイグレーションする[Kelbim](http://kelbim.codenize.tools/)など多くのツールがありますので今後も活用したいと思います。こんなAWS初心者な僕でもInfrastructure as Codeが簡単に実現できて便利だなぁと思う反面、どのような仕組みで動いているのかを理解していかなければならないという危機感も覚えました。
