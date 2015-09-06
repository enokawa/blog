+++
date = "2015-07-29T23:17:11+09:00"
tags = ["nagios", "centos", "aws"]
title = "NagiosのCloudWatchプラグインをつくってみた(PHP)"

+++


## はじめに
タイトルは釣りです。ごめんなさい。笑  
[cloudpack](cloudpack.jp) CTOの[@suz_lab](https://twitter.com/suz_lab)さんがAWS SDK for PHP v1で作成したスクリプトをv2に書き直してみました(本人には了承済みです)。
[前回の記事](/2015/07/27/nagios-nrpe-checklog2/)の続きで、今回はCloudWatchのプラグインを使ってEC2のリソースを監視してみたいと思います。


## 前提条件
* NagiosサーバーににCloudWatch ReadOnly権限のIAM Roleが適用されていること
* phpがインストールされていること
* [Nagiosサーバーの構築](/2015/07/26/nagios-server/)が済んでいること


## AWS SDK for PHPをインストール
まずはじめにAWS SDK for PHP v2をインストールします。インストールする場所はお好きなディレクトリで。
```sh
$ cd /opt/
$ sudo mkdir php-sdk2
$ sudo cd php-sdk2
$ sudo wget https://github.com/aws/aws-sdk-php/releases/download/2.8.14/aws.zip
$ sudo unzip aws.zip
$ sudo wget https://github.com/aws/aws-sdk-php/releases/download/2.8.14/aws.phar
$ sudo vi composer.json  #新しく作成
{
    "require": {
        "aws/aws-sdk-php": "2.*"
    }
}
$ sudo curl -sS https://getcomposer.org/installer | php
$ sudo php composer.phar install
```


## スクリプトの作成
[@suz_labさんのスクリプト](http://blog.suz-lab.com/2011/12/nagioscloudwatchphp.html)をAWS SDK for PHP v2用に書き直します。
```php
$ sudo vim /usr/lib64/nagios/plugins/check_cloudwatch

#!/usr/bin/php
<?php
require_once("/opt/php-sdk2/vendor/autoload.php");

// define status
$ok       = array("code" => 0, "name" => "OK");
$warning  = array("code" => 1, "name" => "WARNING");
$critical = array("code" => 2, "name" => "CRITICAL");
$unknown  = array("code" => 3, "name" => "UNKNOWN");

// set option
$option = getopt("c:w:f:r:n:m:s:u:d:v:");
$critical_size = $option["c"];
$warning_size  = $option["w"];
$region        = $option["r"];
$namespace     = $option["n"];
$metrics       = $option["m"];
$statistics    = $option["s"];
$unit          = $option["u"];
$name          = $option["d"];
$value         = $option["v"];

// set location
use Aws\CloudWatch\CloudWatchClient;
date_default_timezone_set("Asia/Tokyo");

// aws region
$client = CloudWatchClient::factory(array(
    'region' => $region
));

// get metric statistics
@$result = $client->getMetricStatistics(array(
    'Namespace' => $namespace,
    'MetricName' => $metrics,
    'Dimensions' => array(
        array(
            'Name' => $name,
            'Value' => $value,
        ),
    ),
    'StartTime' => strtotime('-10 minutes'),
    'EndTime' => strtotime('now'),
    'Period' => 300,
    'Statistics' => array($statistics),
));

$data = (float)$result["Datapoints"][0]["$statistics"];

// check status
if($data > $critical_size) {
    $status = $critical["code"];
} elseif($data > $warning_size) {
    $status = $warning["code"];
} elseif($data > 0) {
    $status = $ok["code"];
} else {
    $status = $unknown["code"];
}

// output status
$output = " - ${namespace} ${metrics} ${statistics} ${name} ${value}: ${data} ${unit};| data=${data};${warning_size};${critical_size};0;";
switch($status) {
    case $ok["code"]:
        print("CloudWatch " . $ok["name"]       . $output);
        exit($ok["code"]);
    case $warning["code"]:
        print("CloudWatch " . $warning["name"]  . $output);
        exit($warning["code"]);
    case $critical["code"]:
        print("CloudWatch " . $critical["name"] . $output);
        exit($critical["code"]);
    case $unknown["code"]:
        print("CloudWatch " . $unknown["name"]  . $output);
        exit($unknown["code"]);
}
print("CloudWatch " . $unknown["name"] . $output);
exit($unknown["code"]);

$ sudo chmod 755 /usr/lib64/nagios/plugins/check_cloudwatch
```


## 実際にコマンドを打ってテストしてみる
コマンドを打ってみます。
```sh
$ /usr/lib64/nagios/plugins/check_cloudwatch -n AWS/EC2 -m CPUUtilization -s Average -u Percent -d InstanceId -r ap-northeast-1 -w 20 -c 20 -v i-00000001
CloudWatch OK - AWS/EC2 CPUUtilization Average InstanceId i-00000001: 6.656 Percent;| data=6.656;20;20;0;
```


## `commands.cfg`の設定
`check_cloudwatch`コマンドを定義します。
```sh
$ sudo vi /etc/nagios/objects/commands.cfg
define command{
        command_name    check_ec2_cpu
        command_line    $USER1$/check_cloudwatch -n AWS/EC2 -m CPUUtilization -s Average -u Percent -d InstanceId -r ap-northeast-1 -w $ARG1$ -c $ARG2$ -v $HOSTALIAS$
      }
```


## `remotehost.cfg`の設定
前回と同じ要領で`remotehost.cfg`に追記していきます。
```sh
$ sudo vi /etc/nagios/objects/remotehost.cfg
define service{
    use                         generic-service
    host_name                   nagios-agent
    normal_check_interval       3
    service_description         CHECK-CPU
    check_command               check_ec2_cpu!60!80!
}
```

ちゃんと監視できてます！
<img src="/images/nagios-check-cloudwatch.png">


## さいごに
RDSやELBのメトリクスを監視する際に少し工夫が必要になってくるのでその点を改善していきたいです。  
また、前回の記事と同様に、NRPEを使って監視もできるので是非お試しを〜♪


## 参考
* aws/aws-sdk-php at 2.8 https://github.com/aws/aws-sdk-php/tree/2.8
* suz-lab - blog: NagiosのCloudWatchプラグイン(PHP版) http://blog.suz-lab.com/2011/12/nagioscloudwatchphp.html
