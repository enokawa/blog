+++
date = "2016-05-14T18:32:28+09:00"
tags = ["aws", "opsworks", "codecommit"]
title = "OpsWorks + CodeCommitでEC2をプロビジョニングする"
+++

## はじめに
こんにちは、[えのかわ](https://twitter.com/enkw_)です。
今回はOpsWorks(Chef)でEC2インスタンスをプロビジョニングする際にCodeCommitレポジトリを利用してみました。
色々とハマったのでブログで残しておきます。
OpsWorksとCodeCommitについては下記スライドをご参考ください。超ざっくり言うとOpsWorksはAWSに特化したChefサーバで、CodeCommitはGitレポジトリです。

<iframe src="//www.slideshare.net/slideshow/embed_code/key/XmDaDEK7jVFps" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/AmazonWebServicesJapan/aws-opsworks" title="AWS OpsWorksハンズオン" target="blank">AWS OpsWorksハンズオン</a> </strong> from <strong><a href="//www.slideshare.net/AmazonWebServicesJapan" target="blank">Amazon Web Services Japan</a></strong> </div>

<iframe src="//www.slideshare.net/slideshow/embed_code/key/216ZRlQGRhIb8V" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/AmazonWebServicesJapan/aws-black-belt-tech-2015-aws-codecommit-aws-codepipeline-aws-codedeploy" title="AWS Black Belt Tech シリーズ 2015 - AWS CodeCommit &amp; AWS CodePipeline &amp; AWS CodeDeploy" target="blank">AWS Black Belt Tech シリーズ 2015 - AWS CodeCommit &amp; AWS CodePipeline &amp; AWS CodeDeploy</a> </strong> from <strong><a href="//www.slideshare.net/AmazonWebServicesJapan" target="blank">Amazon Web Services Japan</a></strong> </div>


## イメージ
図にするとこのような感じです。間違いがあったらご指摘ください。
<img src="/images/opsworks.png">

1. クライアントPCからCodeCommitレポジトリにChefレシピをpush
2. OpsWorks AgentがCodeCommitレポジトリをpull(更新) -> `update_custom_cookbooks`
3. OpsWorks Agentがプロビジョニング(chef solo)を行う -> `execute_recipes`

## CodeCommit用IAMユーザの作成
CodeCommitレポジトリへソースをプッシュするにはIAMユーザが必要になります。まず始めにCodeCommit専用のIAMユーザを作成します。ポリシーはひとまず`AWSCodeCommitPowerUser`を設定しておきます。
<img src="/images/opsworks1.png">

次にCodeCommitレポジトリにアクセスするためのSSHキーを設定します。`Security Credentials`タブから`Upload SSH public key`をクリックし、公開鍵を貼り付けてください。
<img src="/images/opsworks2.png">

公開鍵の設定が完了するとSSHアクセス用のIDが払い出されます。アクセスキーとは別です。
<img src="/images/opsworks3.png">

最後に`~/.ssh/config`にアクセス情報を記載しておきます。先ほど払い出された`SSH Key ID`と、秘密鍵を指定します。
```
enokawa $ cat ~/.ssh/config
Host git-codecommit.*.amazonaws.com
    User APKAXXXXXXXXXXXXXXX
    IdentityFile "~/.ssh/id_rsa"
```
## CodeCommitレポジトリの作成
3秒で終わります。Repository nameを入力して終わりです。
<img src="/images/opsworks4.png">

作成したレポジトリを選択し、SSH URLをコピーします。
<img src="/images/opsworks5.png">

いざCloneです！
```
enokawa $ git clone ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/enokawa-repo
Cloning into 'enokawa-repo'...
warning: You appear to have cloned an empty repository.
Checking connectivity... done.
enokawa $
enokawa $ ls -l | grep enokawa-repo
drwxr-xr-x   3 enokawa  339809989  102  5 16 00:04 enokawa-repo
```

試しにREADME.mdをpushしてみます。
```
enokawa-repo $ cat README.md
enokawa-repo
===
# OpsWorks + CodeCommitやるぞ！

enokawa-repo $ git add .
enokawa-repo $ git commit -m 'first commit'
[master (root-commit) xxxxxxx] first commit
 1 file changed, 3 insertions(+)
 create mode 100644 README.md
enokawa-repo $ git push origin master
Counting objects: 3, done.
Writing objects: 100% (3/3), 249 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote:
To ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/enokawa-repo
 * [new branch]      master -> master
```
いい感じです！
<img src="/images/opsworks6.png">

適当にApacheをインストールするレシピを書いてプッシュしておきます。
```
enokawa-repo % tree .
.
├── README.md
└── web
    ├── files
    │   └── default
    │       └── index.html
    └── recipes
        └── default.rb

4 directories, 3 files
enokawa-repo % cat web/recipes/default.rb
package ["httpd", "httpd-devel"] do
  action  :install
end

service "httpd" do
  service_name "httpd"
  action  [ :enable, :start ]
  supports  :reload => true
end

cookbook_file "/var/www/html/index.html" do
  not_if { File.exists?("/var/www/html/index.html") }
  source "index.html"
  action :create
  owner "apache"
  group "apache"
  mode "0644"
end
```
## OpsWorksスタックの作成
ガンガンいきます。Add your first stackをクリックします。
<img src="/images/opsworks7.png">

今回はChef 12 stackを作成します。LauchするVPCやKey_pairを指定します。注意する点がRepository URLです。CodeCommitレポジトリを指定する場合、下記のフォーマットとなるようにしてください。SSH Key IDがCodeCommitレポジトリのユーザIDになります。今回の場合だと、`ssh://APKAXXXXXXX@git-codecommit.us-east-1.amazonaws.com/v1/repos/enokawa-repo`といった形になります。 Repository SSH keyには、ペアとなる秘密鍵を入力します。

>ssh://SSH Key ID@CodeCommitレポジトリ名

<img src="/images/opsworks8.png">

Use OpsWorks security groupsがYesのままだと、22番ポートが`0.0.0.0/0`で解放されるのでNoにしておきます。
<img src="/images/opsworks9.png">

これでスタックの作成は完了です。
<img src="/images/opsworks10.png">

## OpsWorksレイヤーの作成
続いてレイヤーの作成です。左側のLayersからAdd layerをクリックします。
<img src="/images/opsworks11.png">

もろもろ入力してAdd layerをクリックします。もちろんSecurityGroupは複数選択できます！
<img src="/images/opsworks12.png">

これでレイヤーの作成は完了です。
<img src="/images/opsworks13.png">

## プロビジョニング対象EC2インスタンスの作成
続いてプロビジョニング対象のEC2インスタンスを作成します。左側のInstancesからインスタンスタイプやEBSボリュームなどの情報を入力してAdd Instanceをクリックします。
<img src="/images/opsworks14.png">

対象のEC2インスタンスを起動します。これでプロビジョニングを行う準備は完了です。
<img src="/images/opsworks15.png">

## プロビジョニング
左側のDeploymentsからRun Commandをクリックします。
<img src="/images/opsworks16.png">

Update Custom Cookbooksを選択し、Update Custom Cookbooksをクリックします。Statusがsuccessfulになったらレポジトリ(Cookbook)の更新は完了です。
<img src="/images/opsworks17.png">

やっとプロビジョニングできます！長かった、、、再度Run CommandからCommandはExecute Recipesを選択し、Recipes to executeに実行したいレシピを入力します。`web::default`と入力していますが、defaultレシピを実行したい場合は`web`でも問題ありません。最後です！Execute Recipesをクリックします！
<img src="/images/opsworks18.png">

ﾔｯﾀｰ！successful!!!!!!
<img src="/images/opsworks19.png">

最後にEC2インスタンスのPublicIPににブラウザでアクセス確認してみます。ﾄﾞｷﾄﾞｷ...
<img src="/images/opsworks20.png">
イヤッッホォォォオオォオウ！

## おわりに
OpsWorks、便利なのですがけっこう手順が多くて疲れました。これからレシピの修正とプロビジョニングを繰り返す場合、下記の手順になります。何度か繰り返したんですけどかなり面倒です。

1. レシピ修正
1. CodeCommitレポジトリにpush
1. OpsWorksで Run Command -> Update Custom Cookbooks
1. OpsWorksで Run Command -> Execute Recipes -> Recipe指定（web::defaultなど）

今後はAWS CLIとか使って作業を効率化したいと思います。