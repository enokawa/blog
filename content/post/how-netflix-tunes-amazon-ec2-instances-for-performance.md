+++
date = "2017-11-29T18:09:58+09:00"
tags = ["aws", "reinvent2017", "ec2" ]
title = "【re:Invent2017レポート】NetflixはパフォーマンスのためにどのようにEC2をチューニングしているか"

+++

## はじめに
こんにちは[えのかわ](https://twitter.com/enkw_)です。下記のセッションレポートです。  
[CMP325 - How Netflix Tunes Amazon EC2 Instances for Performance](https://www.portal.reinvent.awsevents.com/connect/sessionDetail.ww?SESSION_ID=15567)

<iframe width="560" height="315" src="https://www.youtube.com/embed/89fYOo1V2pA" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>

## インスタンスタイプの選定

- Netflix の AWS 環境
  - AutoScalingを採用
- 30 ものインスタンスタイプを使い分けている
  - ファミリーは m4 / c5 /  i3, d2 / r4, x1 / p2, g3, f1
  - medium から 16xlarge(以上) まで
- インスタンス選択のフローチャート
  - <img src="/images/cmp325-1.jpg">
- 全てのインスタンスタイプでロードテスト(スループット計測)を行っている
  - EC2 インスタンスの費用についても計測し、効率の良いインスタンスタイプを選定する
  - インスタンス選定用のツールも自前で作成している
    - <img src="/images/cmp325-2.jpg">

## カーネルチューニング

- Netflix では Ubuntu Linux を利用している
  - 基本的なパフォーマンスチューニングを施しているベースの AMI を持っている
- CPU スケジュール
  - `# schedtool -B PID`
- 仮想メモリ
  - `vm.swappiness = 0`
- Huge pages
  - `# echo madvice > /sys/kernel/mm/transparent_hugepage/enbaled`
- NUMA
  - `kernel.numa_balancing = 0`
- ファイルシステム

```
vm.dirty_ratio = 80
vm.dirty_background_ratio = 5
vm.dirty_expire_centissecs = 12000
mount -o defaults,noatime,discard,nobarrier ...
```

- ストレージ I/O

```
/sys/block/*/queue/rq_affinity   2
/sys/block/*/queue/scheduler     noop
/sys/block/*/queue/nr_requests   256
/sys/block/*/queue/read_ahead_kb 256
mdadm -chunk=64 ...
```

- ネットワーク
```
net.core.somaxconn = 1024
net.core.netdev_max_backlog = 5000
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_wmem = 4096 12582912 16777216
net.ipv4.tcp_rmem = 4096 12582912 16777216
net.ipv4.tcp_max_sync_backlog = 8096
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_porT_range = 10240 65535
net.ipv4.tcp_abort_on_overflow = 1    # maybe
```

- ハイパーバイザー(Xen)
  - `echo tsc > /sys/devices/system/clocksource/clocksource0/current_clockource`

## その他

- EC2 インスタンスの状況を 60 秒以内に分析する
  - [Linux Performance Analysis in 60,000 Milliseconds](https://medium.com/netflix-techblog/linux-performance-analysis-in-60-000-milliseconds-accc10403c55)
- 自前のパフォーマンスモニタリングツール
  - [Vector](https://medium.com/netflix-techblog/introducing-vector-netflixs-on-host-performance-monitoring-tool-c0d3058c3f6f)
  - [Atlas](https://medium.com/netflix-techblog/introducing-atlas-netflixs-primary-telemetry-platform-bd31f4d8ed9a)

## おわりに

レベル高すぎだなというのが率直な意見です。やはり世界的なプロダクトはここまでチューニングやパフォマーンステスト、リソースモニタリングに注力しているんですね。自分自身、カーネルチューニングは苦手な部分なのでこのセッションで初めて聞くパラメータが多かったです。後ほどスライドや動画がアップされるかと思いますので、その際にこのエントリも更新します。セッション動画を観た方がイメージが湧きやすいと思います！
