+++
date = "2017-10-13T23:08:58+09:00"
tags = ["aws", "cloudfront", "s3"]
title = "CloudFront+S3で構成する静的ウェブサイトホスティングのベストプラクティス"

+++

## はじめに
[えのかわ](https://twitter.com/enkw_)です。  
よくある、CloudFrnt + S3 を用いた静的ウェブサイトホスティングの、ベストプラクティスをログとして残します。（よく説明することがあるので、、）

## 構成
S3 を CloudFront Origin として登録する際に、2 つの方法があると思っています。

1. S3 Origin
2. Custom Origin(Static website hostingのURLで)

しかし2つの方法にはメリットデメリットがあります。

- S3 Origin
  - メリット: OAIによるS3への直接アクセスを防ぐことが可能
  - デメリット: ディレクトリを掘ったパスにアクセスした際にindex.htmlがロードされない
- Custom Origin
  - メリット: ディレクトリを掘ったパスにアクセスした際にindex.htmlがロードされる
  - デメリット: S3 へ直接アクセスできてしまう

## 本題
Cloud Front の Origin として、Custom Origin で登録する方法で進めます。該当の S3 は Static website hosting を有効化します。
その上で S3 のバケットポリシー側で UserAgent 値が `Amazon CloudFront` であった場合にコンテンツを返すように記載します。

```json
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "AllowFromCloudFront",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example.com/*",
            "Condition": {
                "StringEquals": {
                    "aws:UserAgent": "Amazon CloudFront"
                }
            }
        }
    ]
}
```

ただこの方法には抜け道があり、下記のように curl コマンドなどで簡単にリクエストヘッダを追加し S3 に直接アクセスできてしまいます。

```sh
$ curl http://example.com.s3-website-ap-northeast-1.amazonaws.com/ --header 'User-Agent:Amazon CloudFront'
```

## さいごに
結果的に、あまりベストプラクティスではないように見えますね、、単一のシングルページであれば S3 Origin として登録する方法で事足りるのですが悩ましいところです。
どなたか良い方法をご存知でしたら教えていただけると喜びます。
