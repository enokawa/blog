+++
date = "2017-11-28T07:17:52+09:00"
tags = ["aws", "reinvent2017", "elasticsearch" ]
title = "【re:Invent2017レポート】ExpediaにおけるAmazonElasticsearchServiceを用いたログ解析"
+++

## はじめに
こんにちは[えのかわ](https://twitter.com/enkw_)です。下記のセッションレポートです。  
[ABD331 - Log Analytics at Expedia Using Amazon Elasticsearch Service](https://www.portal.reinvent.awsevents.com/connect/sessionDetail.ww?SESSION_ID=16756)

<iframe width="560" height="315" src="https://www.youtube.com/embed/oJUpUQ_yNVw" frameborder="0" allowfullscreen></iframe>

## Amazon Elasticsearch Service について
まず AWS の Principal Product Manager である [Carl Meadows](https://www.portal.reinvent.awsevents.com/connect/speakerDetail.ww?PERSON_ID=30D17B8F99B3E8399A8147EA4A7CB6A2&tclass=popup) 氏から簡単に Elasticsearch と Amazon Elasticsearch Service(以下ES)についての説明がありました。

### 現在、機械的に生成されたデータは多い

- 手動による IT から DevOps への移行
- IoT デバイス
- クラウドベースのアーキテクチャ

### Elasticsearch のベネフィット

- オープンソース
- 素早い価値創出

### [ELK Stack](http://www.elastic.co/webinars/introduction-elk-stack) とは下記の 3 つをまとめたもの

- Elasticsearch
- Logstash
- Kibana

### Elasticsearch のユースケース

- アプリケーションモニタリング と Root-cause Analysis(根本原因解析)
- Security Information and Event Management(SIEM)
- IoT & モバイル
- ビジネス & クリックストリーム分析

### ES について

- マネージド Elasticsearch + Kibana
- ベネフィット
  - オープンソース
  - 簡単
  - スケーラブル
  - セキュア
  - 高可用性
  - 他 AWS サービスとの連携

### [事例](https://aws.amazon.com/jp/elasticsearch-service/)

  - Adobe
  - Netflix など

## Expedia における ES のユースケース

Expedia の [Kuldeep Chowhan](https://www.portal.reinvent.awsevents.com/connect/speakerDetail.ww?PERSON_ID=5266D16D84B66AB37CDF8C69E92B7631&tclass=popup)([@this_is_kuldeep](https://twitter.com/this_is_kuldeep)) 氏から Expedia のユースケースについて発表がありました。

### 使用量

- 150 以上の ES クラスタ
- 450 台以上の EC2
- 300 万以上の ドキュメント
- 30TB 以上のデータ

### なぜ ES を選んだか

- セットアップが容易
- 高可用性
- セキュア
  - [Elasticsearch access policy](http://docs.aws.amazon.com/ja_jp/elasticsearch-service/latest/developerguide/es-createupdatedomains.html#es-createdomain-configure-access-policies) で権限や接続元 IP などの制限が可能
- ディスクの拡張/タイプ変更が可能
- CloudWatch でのモニタリング & バックアップの作成が容易

### ログ解析のアーキテクチャ
- Docker startup logs to Elasticsearch Docker の起動ログ解析
  - `ECS(agent / log driver) -> Docker(log_driver) -> fluentd -> ES <- Kibana`
- AWS CloudTrail のログ解析
  - `CloudTrail -> S3 -> SNS -> Lambda -> ES <- Kibana`
  - Github にテンプレートを公開している: [ExpediaDotCom/cloudtrail-log-analytics](https://github.com/ExpediaDotCom/cloudtrail-log-analytics)
- CI/CD プラットフォームの KPI 解析
  - `ruby app / node.js app -> SNS -> Lambda -> ES <- node.js app`
- 分散トレーシングのプラットフォーム解析
  - `Microservices -> java library -> Kinesis <- java app -> kafka <- java app -> ES <- nodejs app`

### まとめ

- ES Cluster がスケールすると同時にデータも同期される
- クラスタの監視と最適化が可能
- Elasticsearch のバージョンアップグレードができない
  - 手動での移行が必要
- ディスクをどれだけ利用しているかがメトリクスから読み取りづらい
- ベネフィット
  - セットアップが容易
  - 高可用性
  - セキュア
  - モニタリング&バックアップ可能

## おわりに

実は僕はまだ Elasticsearch や ES を使ったことがないので何とも言えないのですが、単にログを可視化するだけではなくそのログからどのような問題を見出す&解決するかが重要だと感じました。当たり前かもですが。とりあえず触ってみます。あと[クラスメソッドさんの記事も出てました](https://dev.classmethod.jp/cloud/aws/loganalytics-at-expedia-using-amazones/)。勉強になります。
