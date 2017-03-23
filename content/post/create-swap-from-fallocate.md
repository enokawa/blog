+++
date = "2016-09-14T21:46:09+09:00"
tags = ["linux"]
title = "fallocateでswapを作成する"

+++

## はじめに
こんにちは、[えのかわ](https://twitter.com/enkw_)です。  
fallocateでswapを作成する手順を残しておきます。OSはAmazonLinuxです。

## ざっくり
3GB分のswapを作成します。
```
[root@test-01 ~]# mkdir /var/lib/swap
[root@test-01 ~]# cd /var/lib/swap/  
[root@test-01 swap]# fallocate -l 3g swapfile.0
[root@test-01 swap]# mkswap ./swapfile.0
Setting up swapspace version 1, size = 3145724 KiB
no label, UUID=c7a98a35-47a1-41dc-ac86-05ad1910f4ca
[root@test-01 swap]# chmod 600 swapfile.0
[root@test-01 swap]# swapon swapfile.0
[root@test-01 swap]# free -m | grep Swap
Swap:         3071          0       3071
[root@test-01 swap]# vim /etc/fstab
/var/lib/swap/swapfile.0 swap       swap defaults 0  0
[root@test-01 swap]# cat /proc/sys/vm/swappiness
60
```

## さいごに
以下の記事が参考になりました。  
[Azure仮想マシンへのSwap領域の割り当て](http://qiita.com/unosk/items/984b336a292fc1dcf03c)
