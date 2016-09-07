+++
date = "2016-09-07T21:00:37+09:00"
tags = ["aws", "route53", "iam"]
title = "特定のRoute53のHostedZoneのみの操作権限を与えたい"
+++

## はじめに
こんにちは、[えのかわ](https://twitter.com/enkw_)です。  
特定のRoute53のHostedZoneのみを操作できるIAMユーザを作成したい場面があります。  
そのIAMポリシーを作成したのでメモです。

## HostedZoneの作成
まずRoute53が操作できる環境で２つのHostedZoneを作成します。
<img src="/images/hostedzone.png">

## IAMユーザの作成
ポリシーは下記です。  
`<HostedZoneID>`に、そのIAMユーザに操作させたいHotedZoneのIDを記載します。  
今回は enokawa.me のHostedZoneIDを記載しました。
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:ChangeResourceRecordSets",
                "route53:GetHostedZone",
                "route53:ListResourceRecordSets"
            ],
            "Resource": "arn:aws:route53:::hostedzone/<HostedZoneID>"
        },
        {
            "Effect": "Allow",
            "Action": [
                "route53:GetChange"
            ],
            "Resource": "arn:aws:route53:::change/*"
        }
    ]
}
```

## 検証
作成したIAMユーザでAWSマネージメントコンソールにログインし、Route53にアクセスします。  
権限エラーがでますね。
<img src="/images/route53-error1.png">

HostedZone一覧も閲覧できません。
<img src="/images/route53-error2.png">

下記の形式のURLでアクセスする必要があります。
```
https://console.aws.amazon.com/route53/home?region=ap-northeast-1#resource-record-sets:<HostedZoneID>
```

enokawa.me のHostedZoneIDを指定してアクセスしてみましょう。
<img src="/images/recordlist1.png">
わぁい

enokawa.org のHostedZoneIDを指定してアクセスすると何も表示されません。  
想定通りの挙動です。
<img src="/images/recordlist2.png">
以下の警告文が表示されます。

> r53_user is not authorized to perform: route53:GetHostedZone on resource: hostedzone/HostedZoneID

## ALIASレコードは設定できるのか
前述したIAMポリシーにはS3やELB、CloudFrontなどの参照権限が記載されていないので、Alias Targetに候補は出てきません。
<img src="/images/alias_target.png">

が、存在するELBのエンドポイントをペーストしてレコードを登録してみると、、、
<img src="/images/alias_record.png">
お？

<img src="/images/recordlist3.png">
わぁい

レコードを操作する権限があるのでALIASレコードの登録は可能です。  
仮にそのユーザにALIASレコードを登録させたい場合はELBのエンドポイントを教えてあげましょう。

## おわりに
IAM職人の朝は早い。
