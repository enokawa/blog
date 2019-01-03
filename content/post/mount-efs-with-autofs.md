+++
date = "2018-06-13T23:02:51+09:00"
tags = ["aws", "ec2", "autofs"]
title = "autofsでEFSをマウントする"

+++

## はじめに

こんにちは[えのかわ](https://twitter.com/enkw_)です。EFS、[東京リージョン対応がアナウンスされました](https://aws.amazon.com/jp/blogs/news/amazon-elastic-file-system-efs-nrt/)ね。  
リリース前にちょいと検証してみます。単純にマウントするだけではつまらないので [autofs](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/storage_administration_guide/nfs-autofs) でマウントしてみます。

## AWS リソース作成
こんな感じで作成しました。リージョンはオレゴンでやってます。OS は Amazon Linux です。
<img src="/images/efs.png">

## EC2設定

autofs をインストールしてマウントポイント用のディレクトリを作成します。
```sh
[ec2-user@ip-172-31-38-110 ~]$ sudo yum install autofs
[ec2-user@ip-172-31-38-110 ~]$ mkdir /exports
```

続いて autofs の設定です。
```sh
[ec2-user@ip-172-31-38-110 ~]$ sudo sh -c "echo '/- /etc/auto.nfs --timeout=3' >> /etc/auto.master"
[ec2-user@ip-172-31-38-110 ~]$ cat /etc/auto.nfs
/exports -nfsvers=4.1 us-west-2a.fs-xxxxxxxx.efs.us-west-2.amazonaws.com:/
```

autofs 起動します。
```sh
[ec2-user@ip-172-31-38-110 ~]$ sudo /etc/init.d/autofs start
Starting automount:                                        [  OK  ]
```

`/exports` ディレクトリに移動するとマウントされていることが確認できました。
```sh
[ec2-user@ip-172-31-38-110 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        484M   56K  484M   1% /dev
tmpfs           494M     0  494M   0% /dev/shm
/dev/xvda1      7.8G  1.1G  6.7G  14% /
[ec2-user@ip-172-31-38-110 ~]$ cd /exports/
[ec2-user@ip-172-31-38-110 exports]$ df -h
Filesystem                                            Size  Used Avail Use% Mounted on
devtmpfs                                              484M   56K  484M   1% /dev
tmpfs                                                 494M     0  494M   0% /dev/shm
/dev/xvda1                                            7.8G  1.1G  6.7G  14% /
us-west-2a.fs-0dee8ea4.efs.us-west-2.amazonaws.com:/  8.0E     0  8.0E   0% /exports
[ec2-user@ip-172-31-38-110 exports]$
[ec2-user@ip-172-31-38-110 exports]$ cat /proc/mounts  | grep exports
/etc/auto.nfs /exports autofs rw,relatime,fd=19,pgrp=29007,timeout=3,minproto=5,maxproto=5,direct 0 0
us-west-2a.fs-xxxxxxxx.efs.us-west-2.amazonaws.com:/ /exports nfs4 rw,relatime,vers=4.1,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=172.31.38.110,local_lock=none,addr=172.31.49.104 0 0
```

## おわりに
autofs を利用することで `/etc/fstab` を修正せずに済んで最高ですね。Chef や Ansible で構成管理しやすいので便利です。  
EFS の欠点として、バックアップの機能が無いことが挙げられますので、別途そこは作り込む必要ありますね。既に実装している記事も見かけます。  
EFS の標準機能としてバックアップ/リストアの機能がリリースされることを待ちましょう。
