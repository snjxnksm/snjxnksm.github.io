---
layout: post
title: "Docker環境でRailsのステップ実行するには"
date: "2019-05-31 17:07:26 +0900"
tags:
  - ruby
  - rails
  - docker
---

docker-compose.ymlに以下、`stdin_open: true`と` tty: true`を追加して、コンテナを再起動する。

```
version: '3'
services:
  #
  # Railsウェブアプリ
  #
  rails_contener:
    ・
    ・
    ・
    stdin_open: true
    tty: true
```

ターミナルを余分に開き、rails_contenerへアタッチする。

```
$ docker attach rails_contener
```

アタッチした直後は特に何も出力されないはず。Web画面を操作すると、実行ログがアタッチプロセスの方にも出力される。
ソースコード中に、 `require 'byebug'; byebug` と１行だけ書いておくと、そこで止めてくれる。

アタッチしたプロセスから抜ける場合は、ctrl + p と ctrl + q を連続で打つと、コンテナを停止させずに抜け出せる。
（ここで ctrl + c とかやると、rails_contenerサービス自体が死んでしまうので注意！）
