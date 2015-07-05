+++
date = "2014-01-22T00:28:28+09:00"
tags = ["heroku"]
title = "【備忘録】herokuへのデプロイ"

+++

備忘録として、ローカルで作成したrubyプログラムを  
herokuへデプロイしたのでそのログを。

環境  
MacBook Pro 13inch (Mid 2012)  
OS X 10.8.5  
iTerm2  




まずはheroku-toolbeltなるものをインストール

次にherokuをインストール

`sudo gem install heroku`

次にherokuにログイン

`heroku login`

次にSSH鍵を作り直す

`heroku keys:clear`  
`heroku keys:add`

次に簡単なsinatraアプリケーションを作成します。


```
# app.rb
require 'sinatra'

get '/' do
   "Hello World"
end
```

```
# Gemfile
source 'https://rubygems.org'
gem 'sinatra'
```

```
# Procfile
web: bundle exec backup config.ru -p $PORT
```

```
# config.ru
$:.unshift(File.dirname(__FILE__))

require "sample"

run Sample
```

```
# .gitignore
.bandle
```

ローカルで起動

`foreman start`

デプロイ

```
git init
git add *
git commit -m 'first commit'
heroku create
git remote -v
git push  -u heroku master
heroku open
```

参考サイト  
i2bsの日記  
http://i2bskn.hateblo.jp/entry/2013/06/11/000625

雑な書き方でスミマセン。
