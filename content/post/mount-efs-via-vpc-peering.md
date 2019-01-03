+++
date = "2018-06-14T23:02:51+09:00"
tags = ["aws", "ec2", "efs"]
title = "EFSをVPCペアリング経由でマウントする"

+++

## はじめに

こんにちは[えのかわ](https://twitter.com/enkw_)です。前回は単純に [autofs で EFS をマウント](/2018/06/13/mount-efs-with-autofs/)しましたが、今回は VPC Peering経由でマウントしてみたいと思います。
AWS のドキュメントには、VPN や VPC Peering 経由ではマウントできないとの記述があります。

>AWS Direct Connect を使用して、オンプレミスのデータセンターサーバーから Amazon EFS ファイルシステムをマウントできます。ただし、VPN 接続や VPC ピア接続などの他の VPC プライベート接続機能はサポートされていません。

https://docs.aws.amazon.com/ja_jp/efs/latest/ug/limits.html

クラメソさんの記事を参考にさせてもらいました。  
[SSH ポートフォワーディングを使って EFS を Mac からマウントしてみた](https://dev.classmethod.jp/cloud/aws/mount-efs-over-ssh/)

## AWS リソース作成
こんな感じで作成しました。東京リージョンからオレゴンリージョンの EC2 を経由して EFS をマウントします。RouteTable や SecurityGroup の設定はお互い通信できるように事前に済ませておきましょう。
<img src="/images/efs2.png">

## EC2設定(東京側)

オレゴン側の設定は特に必要ありません。こういう感じで東京の EC2 からオレゴンの EC2 に SSH できるように設定しておきます。
```sh
[ec2-user@ip-10-0-0-79 ~]$ ssh -i .ssh/enokawa-kensho-us-west-2.pem ec2-user@172.31.38.154
Last login: Thu Jun 14 03:40:07 2018 from 10.0.0.79

       __|  __|_  )
       _|  (     /   Amazon Linux AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-ami/2018.03-release-notes/
11 package(s) needed for security, out of 13 available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-172-31-38-154 ~]$
```

ポートフォワードしてみます。 `-fN` オプションを用いてバックグラウンドでプロセスを動かし続けます。
```sh
[ec2-user@ip-10-0-0-79 ~]$ ssh -i .ssh/enokawa-kensho-us-west-2.pem -fN -L 2049:us-west-2a.fs-xxxxxxxx.efs.us-west-2.amazonaws.com:2049 ec2-user@172.31.38.154
[ec2-user@ip-10-0-0-79 ~]$ ps aux | grep "ssh -[i]"
ec2-user  2839  0.0  0.0 173520   884 ?        Ss   03:47   0:00 ssh -i .ssh/enokawa-kensho-us-west-2.pem -fN -L 2049:us-west-2a.fs-xxxxxxxx.efs.us-west-2.amazonaws.com:2049 ec2-user@172.31.38.154
```

マウントしてみます。
```sh
[ec2-user@ip-10-0-0-79 ~]$ sudo mkdir /exports
[ec2-user@ip-10-0-0-79 ~]$ sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 localhost:/ /exports
[ec2-user@ip-10-0-0-79 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        484M   60K  484M   1% /dev
tmpfs           494M     0  494M   0% /dev/shm
/dev/xvda1      7.8G  1.1G  6.7G  14% /
localhost:/     8.0E     0  8.0E   0% /exports
```

マウントできました。ただ通常のマウントよりも少しラグはあります。ポートフォワーディングかつリージョンまたいでるのでしょうがないですね。

## おわりに
オンプレミス環境などから EFS をマウントする場合は DirectConnect 経由の方がよさそうです。
