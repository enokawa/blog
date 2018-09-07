+++
date = "2017-03-23T14:17:24+09:00"
tags = ["aws", "autoscaling", "ssm", "cloudwatch"]
title = "AutoScalingからCloudWatchEvents(SSM)を呼ぶ"

+++

## はじめに
ご無沙汰してます、[えのかわ](https://twitter.com/enkw_)です。  
先日、[CloudWatch Events の Target に SSM Run Command が追加されました](https://aws.amazon.com/jp/blogs/aws/ec2-run-command-is-now-a-cloudwatch-events-target/)。  
今回は、Auto Scaling の Lifecycle Hooks の `TERMINATING` をトリガーに、CloudWatch Events から SSM Run Commnad を呼んでみます。`LAUNCHING` に関しては、特に SSM を呼ぶ必要がなさそうなので、 User data を用いて初期設定を行います。  

<img src="/images/as-flow.png">


## AutoScaling設定
ざっと aws cli を流します。

#### Launch Configuration作成
ベースとなる AMI には SSM Agent を導入している必要があります。今回の OS は AmazonLinux です。
```sh
$ aws autoscaling create-launch-configuration \
--launch-configuration-name test-web-lc \
--image-id ami-xxxxxx \
--key-name enokawa-kensho \
--security-groups sg-xxxxxxxx sg-xxxxxxxx \
--user-data file://userdata.txt \
--instance-type t2.micro \
--block-device-mappings "[{\"DeviceName\": \"/dev/xvda\",\"Ebs\":{\"VolumeSize\":100,\"VolumeType\":\"gp2\"}}]" \
--iam-instance-profile web \
--associate-public-ip-address

$ cat userdata.txt
#!/bin/bash

# touch contents
touch /var/www/html/index.html

# complete auto scaling lifecycle
INSTANCE_ID=`/usr/bin/curl -s 169.254.169.254/latest/meta-data/instance-id`
aws autoscaling complete-lifecycle-action \
--lifecycle-hook-name test-web-lh-launching \
--auto-scaling-group-name test-web-asg \
--lifecycle-action-result CONTINUE \
--instance-id ${INSTANCE_ID}
```

### AutoScaling Group作成
Desired は 2 台、最高 4 台までスケールアウトします。  
下記コマンド実行後には 2 台起動します。
```sh
$ aws autoscaling create-auto-scaling-group \
--auto-scaling-group-name test-web-asg \
--launch-configuration-name test-web-lc \
--min-size 2 \
--max-size 4 \
--desired-capacity 2 \
--default-cooldown 300 \
--availability-zones "ap-northeast-1a" "ap-northeast-1c" \
--load-balancer-names enokawa-me \
--health-check-type EC2 \
--health-check-grace-period 300 \
--vpc-zone-identifier subnet-xxxxxxx,subnet-xxxxxxx \
--tags ResourceId=test-web-asg,ResourceType=auto-scaling-group,Key=Name,Value=test-web,PropagateAtLaunch=true
```

### Scaling Policies設定
こんな感じです。  
ELB(enokawa-me) のリクエスト数が 100 以上なら 2 台追加、30 以下なら 2 台削除します。

<img src="/images/scaling-policies.png">

### Lifecycle hook作成
```sh
aws autoscaling put-lifecycle-hook \
--lifecycle-hook-name test-web-lh-launching \
--auto-scaling-group-name test-web-asg \
--lifecycle-transition autoscaling:EC2_INSTANCE_LAUNCHING \
--heartbeat-timeout 1800 \
--default-result ABANDON

$ aws autoscaling put-lifecycle-hook \
--lifecycle-hook-name test-web-lh-terminating \
--auto-scaling-group-name test-web-asg \
--lifecycle-transition autoscaling:EC2_INSTANCE_TERMINATING \
--heartbeat-timeout 1800 \
--default-result ABANDON
```

### CloudWatch Events 作成
いよいよ CloudWatch Events 作成です。
<img src="/images/cloudwatch-rule.png">
コマンドはこんな感じです。S3 に syslog を転送して、complete-lifecycle-action コマンドを実行後にサービスアウトする流れです。
```json
{
  "commands": [
    "INSTANCE_ID=`/usr/bin/curl -s 169.254.169.254/latest/meta-data/instance-id`",
    "aws s3 cp /var/log/messages s3://enokawa-logs/${INSTANCE_ID}/",
    "aws autoscaling complete-lifecycle-action --lifecycle-hook-name test-web-lh-terminating --auto-scaling-group-name test-web-asg --lifecycle-action-result CONTINUE --instance-id ${INSTANCE_ID}"
  ],
  "executionTimeout": [
    "600"
  ]
}
```

## AutoScaling実施
俺たちの ApacheBench で ELB に対してリクエストを送ります。
```sh
$ ab -c 100 -n 100 http://enokawa-me-0123456789.ap-northeast-1.elb.amazonaws.com/
```

`Pending` -> `Pending:Wait` -> `Pending:Proceed` -> `InService` の順番で ELB に Service In しました。  
スケールインのアラームが発砲するまでしばらく待ちます。
<img src="/images/scaleout.png">

`Terminating` -> `Terminating:Wait` -> `Terminating:Proceed` -> `Terminated` の順番でサービスアウトしました。
<img src="/images/scalein.png">

s3 の中身を見てみると、、、
```sh
as % aws s3 ls s3://enokawa-logs/
                           PRE i-xxxxxxxxxxxxxx1/
                           PRE i-xxxxxxxxxxxxxx2/
                           PRE i-xxxxxxxxxxxxxx3/
                           PRE i-xxxxxxxxxxxxxx4/
```
あれ？ 4 台分のログが転送されてますね。  
Run Command の履歴をみてみると、4 台に対して `AWS-RunShellScript` が実行されたようです。  
僕の妄想では、`Terminating:Wait` となっていて、かつ特定のタグを付与されているインスタンスに対して Run Command が実施されるはずでした。  
CloudWatch Events の Event Source と Targets は別で考えた方がよさそうです。

ということで、AutoScaling で `Terminating:Wait` となっているインスタンス**のみ**に SSM Run Command を実施する場合は Lambda を利用する必要がありそうです。  
もっと良い方法をご存知の方は教えてください。。。Command で判断するパターンもありですかねぇ。

## おわりに
Lambda 作成する手間が省けたぞﾔｯﾀｰと喜んでいたのも束の間でした、、、  
`Terminating:Wait` となっている instance-id を拾って、そのインスタンスに対して Run Command を実行する Lambda を書きたいと思います。できたらブログもかきます。
