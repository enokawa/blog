+++
date = "2017-04-23T16:02:58+09:00"
tags = ["aws", "autoscaling", "ssm", "cloudwatch", "lambda"]
title = "AutoScalingからCloudWatchEvents(Lambda)を呼ぶ"

+++

## はじめに
ご無沙汰してます、[えのかわ](https://twitter.com/enkw_)です。[先日投稿したエントリ](/2017/03/23/use-auto-scaling-lifecycle-hook-with-cloudwatch-events-ssm/)の続きです。  
今回は、Auto Scaling の Lifecycle Hooks の `TERMINATING` をトリガーに、CloudWatch Events から Lambda Function をキックして、SSM Run Commandを呼んでみます。  
ゴールはログを S3 に転送することとします。

<img src="/images/as-scalein-flow.png">

## AutoScaling設定
[前回のエントリ](/2017/03/23/use-auto-scaling-lifecycle-hook-with-cloudwatch-events-ssm/)と同様の AWS CLI を流します。  
Scaling Policies、Lifecycle hook の作成も併せて行います。

### Lambda Function作成
今回はせっかくなので [Serverless Framework](https://serverless.com/) を用いて Lambda Function を作成します。
<script src="https://gist.github.com/enokawa/9f0d4b30924f5d1cdeba3a7675aecca8.js"></script>

## CloudWatch Events作成
CloudWatch Events の設定も Serverless Framework で行いたかったのですが何故か上手くいかず、、、しょうがなく GUI で設定しました。
ドキュメントは[こちら](https://serverless.com/framework/docs/providers/aws/events/cloudwatch-event/)です。

<img src="/images/cloudwatch-events-as.png">

## AutoScaling実施
今回は Scale In Event のみ Execute します。

`Terminating` -> `Terminating:Wait` -> `Terminating:Proceed` -> `Terminated` の順番でサービスアウトしました。
<img src="/images/cloudwatch-event-scalein.png">

s3 の中身を見てみると、、、
```sh
sls-autoscaling % aws s3 ls s3://enokawa-logs/
                           PRE i-xxxxxxxxxxxxxx1/
                           PRE i-xxxxxxxxxxxxxx2/
```
問題なくスケールインされたインスタンスのログのみ転送されてました。

## おわりに
[awslabs のサンプルスクリプト](https://github.com/awslabs/aws-lambda-lifecycle-hooks-function/blob/master/lambda_backup.py)が参考になりました。  
AutoScaling の Complete Lifecycle Action を Lambda で実施するか Run Command で実施するかどうかは賛否両論あるかと思いますが、今回はシンプルに実行できる Run Command で投げてみました。  
Lifecycle Hooks 便利ですね〜。
