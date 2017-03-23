+++
date = "2016-10-06T17:14:37+09:00"
tags = ["aws", "rds"]
title = "RDSのDB Engineのバージョン一覧をAWS CLIで確認する"
+++

## はじめに
こんにちは、[えのかわ](https://twitter.com/enkw_)です。  
RDSのDB Engineのバージョン一覧をAWS CLI一発で確認する方法のメモです。

## さっそく
MySQLのバージョン一覧を確認してみます。
```sh
$ aws rds describe-db-engine-versions --engine mysql --query 'DBEngineVersions[].EngineVersion' --output table
--------------------------
|DescribeDBEngineVersions|
+------------------------+
|  5.5.40                |
|  5.5.40a               |
|  5.5.40b               |
|  5.5.41                |
|  5.5.42                |
|  5.5.46                |
|  5.6.19a               |
|  5.6.19b               |
|  5.6.21                |
|  5.6.21b               |
|  5.6.22                |
|  5.6.23                |
|  5.6.27                |
|  5.6.29                |
|  5.7.10                |
|  5.7.11                |
+------------------------+
```

## Auroraの場合
Auroraの場合は単純に`--engine`パラメータを修正するのみです。
```sh
$ aws rds describe-db-engine-versions --engine aurora --query 'DBEngineVersions[].EngineVersion' --output table
--------------------------
|DescribeDBEngineVersions|
+------------------------+
|  5.6.10a               |
+------------------------+
```

## さいごに
いちいちAWSマネージメントコンソールで調べるのも面倒なので便利ですね、AWS CLI。  
これからもお世話になりそうです。
