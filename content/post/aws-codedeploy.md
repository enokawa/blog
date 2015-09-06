+++
date = "2015-09-01T20:56:17+09:00"
tags = ["aws", "codedeploy"]
title = "AWS CodeDeployを使ってでデプロイを自動化してみた"

+++

## はじめに
こんにちは、インフラエンジニア見習いの[えのかわ](https://twitter.com/enkw_)です。
今回は、AWS CodeDeployを使ってデプロイを自動化してみたのでログを残しておきます。
CodeDeployアプリケーションの作成は省略します。
下記のURLがとても参考になりました！

* http://www.ryuzee.com/contents/blog/7022
* http://dev.classmethod.jp/cloud/codedeploy-ataglance/


## イメージ
図にするとこのような感じです。間違いがあったらご指摘ください。
<img src="/images/aws-codedeploy1.png">

1. クライアントPCからプロジェクトを圧縮してS3にアップロード
2. クライアントからCodeDeployのAPIを叩いてEC2(CodeDeploy Agent)にメタデータを渡す
3. CodeDeploy AgentがS3にポーリングする
4. S3から圧縮されたファイルをダウンロードして展開する


## AppSpec.ymlの作成
`appspec.yml`は以下のような形です。AppSpecについては、[クラスメソッドさんの記事](http://dev.classmethod.jp/cloud/aws/code-deploy-appspec/)が大変参考になりました。
```
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html/
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies
      timeout: 300
      runas: root
    - location: scripts/start_server
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server
      timeout: 300
      runas: root
```


## CodeDeployでプロジェクトをデプロイするには
[@ryuzee](http://twitter.com/ryuzee)さんの[ブログ](http://www.ryuzee.com/contents/blog/7022)にも紹介されている通り、最終的には以下のコマンドを打たなければデプロイができません。(ポチポチでデプロイできるのかな？)

`deploy push`コマンドでプロジェクトのファイルをzipにしてS3にアップロードします。
```sh
$ aws deploy push \
>  --application-name CodeDeployTest \
>  --s3-location s3://enokawa-test-bucket/app.zip \
>  --source ./ \
>  --region ap-northeast-1
To deploy with this revision, run:
aws deploy create-deployment --application-name CodeDeployTest --s3-location bucket=enokawa-test-bucket,key=app.zip,bundleType=zip,eTag="XXXXXXXXXXXXXXXXXXXXXXXXX" --deployment-group-name <deployment-group-name> --deployment-config-name <deployment-config-name> --description <description>
```

`deploy create-deployment`コマンドでDeploymentGroupのEC2にデプロイする。
```sh
$ aws deploy create-deployment \
>   --application-name CodeDeployTest \
>   --s3-location bucket=enokawa-test-bucket,key=key=app.zip,bundleType=zip,eTag="XXXXXXXXXXXXXXXXXXXXXXXXX" \
>   --deployment-group-name CodeDeployTest \
>   --region ap-northeast-1
{
    "deploymentId": "d-XXXXXXXXX"
}
```


## シェルスクリプトの作成
毎回このコマンドを打つのは面倒なのでシェルスクリプトにしてみました。
例外処理などは書いていないです。ツッコミなどある方はgistに直接コメントするか、[Twitter](https://twitter.com/enkw_)でメンション下さると嬉しいです！
<script src="https://gist.github.com/enokawa/ff873740c702e5dd6634.js"></script>


## デプロイの流れ
以下の流れでデプロイができたら理想的だなと思いました。

1. ファイルを修正する
2. ローカルで正しく修正されているかを確認する
3. Gitでコミット/プッシュする
4. `deploy.sh`実行
5. 反映されているか確認


## おわりに
AWS CodeDeployを用いることで、今までのデプロイ作業が少し楽になるかもしれません。Githubとの連携も可能らしいので、今後試してみたいと思います。
