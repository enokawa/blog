+++
date = "2018-01-11T22:18:02+09:00"
tags = ["aws", "workspaces", "directoryservice", "radius"]
title = "GoogleAuthenticatorを利用してAmazon WorkSpacesへ多要素認証ログインする"
+++

## はじめに
[えのかわ](https://twitter.com/enkw_)です。  
Amazon WorkSpaces に多用素認証ログインするのに詰まったのでエントリ書きます。

## 構成
ざっくりとこんな感じです。現時点(2018/01/12)で Simple AD だと MFA を利用した上での WorkSpaces ログインができないので Microsoft AD を使用しています。
<img src="/images/workspaces.png">

## 手順
VPC の作成とかは省きます。

### DirectoryServiceの作成
作成には 20 分 〜 30 分ほど掛かります。
```sh
$ aws ds create-microsoft-ad \
--name ad.enokawa.co \
--password xxxxxxxxxxxxxx \
--vpc-settings VpcId=vpc-xxxxxxx,SubnetIds=subnet-xxxxxxx,subnet-xxxxxxx
{
    "DirectoryId": "d-xxxxxxxxx"
}
```

Directory の Status が Active になったら [Register](https://docs.aws.amazon.com/ja_jp/workspaces/latest/adminguide/register-deregister-directory.html) します。(Register する API が見つけられなかったけどないのかな、、、  

### WorkSpaceの作成
CLI で作成する場合は WorkSpace 用のユーザが DirectoryService に存在している必要があります。DirectoryService にユーザを作成する場合は少し面倒なので[公式ドキュメント](https://docs.aws.amazon.com/ja_jp/workspaces/latest/adminguide/launch-workspace-simple-ad.html)を参考にポチポチで作成します。  

`お客様の Amazon WorkSpace ( Your Amazon WorkSpace )` という件名でメールが AWS 側から送信されるので、メール本文に記載されている手順通りに進めます。専用のクライアントでログインを試みると、ユーザ名とパスワードが表示されます。これで普通にログインできます。
<img src="/images/workspaces-logn.png" width=50% height=50%>

### RADIUSインスタンスの設定
gist はります。`hostnamectl` コマンドでホスト名を設定しておいてください。
<script src="https://gist.github.com/enokawa/330a96ca42a162bed0ec06e78d12640c.js"></script>

### Microsoft ADでのRADIUS有効化
RADIS インスタンスの SecurityGroup で Microsoft AD からの UDP 1812 ポート通信を許可しておきます。
```sh
$ cat radius.json
{
  "RadiusServers": ["10.0.0.xxx"],
  "RadiusPort": 1812,
  "RadiusTimeout": 10,
  "RadiusRetries": 2,
  "SharedSecret": "xxxxxxxxxxxxxxxxxxx",
  "AuthenticationProtocol": "PAP",
  "DisplayLabel": "passcode",
  "UseSameUsername": true
}
$
$ aws ds enable-radius \
--directory-id d-xxxxxxxxx \
--radius-settings file://radius.json
$
```

`RadiusStatus` が Completed になっていれば OK です。
```sh
$ aws ds describe-directories --directory-ids d-xxxxxxxxx \
--query 'DirectoryDescriptions[].RadiusStatus[]' --output text
Completed
```

再度、WorkSpace にログインを試みると MFA コードの入力欄がでてきます。コードを入力してログインできれば OK です。
<img src="/images/workspaces-logn-mfa.png" width=50% height=50%>

## おわりに
AWS のブログの方が分かりやすいかもです。  
[Google Authenticator を使って Amazon WorkSpaces に多要素認証ログイン](http://aws.typepad.com/sajp/2014/10/google-authenticator.html)  
[CentOS 7 Minimal & Two factor Authentication using FreeRADIUS 3, SSSD 1.12, & Google Authenticator](https://github.com/rharmonson/richtech/wiki/CentOS-7-Minimal-&-Two-factor-Authentication-using-FreeRADIUS-3,-SSSD-1.12,-&-Google-Authenticator)
