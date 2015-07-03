+++
date = "2014-01-06T02:07:57+09:00"
tags = [ "git", "github" ]
title = "【備忘録】githubへのコミット"
+++

gitのローカルリポジトリからgithubへのコミット方法

環境
MacBook Pro 13inch (Mid 2012)
OS X 10.8.5
iTerm2


まずローカルディレクトリに移動

`cd git/hoge/`

git 開始

`git init`

フォルダ以下全部追加

`git add *`

ローカルにコミット

`git commit -m 'first commit'`

リモートレポジトリ originを追加

`git remote add origin https://github.com/user/hoge.git`

githubにpush

`git push origin master`


参考にしたサイト

goryugo Hatena Diary  
http://d.hatena.ne.jp/goryugo/20081026/1225007428
