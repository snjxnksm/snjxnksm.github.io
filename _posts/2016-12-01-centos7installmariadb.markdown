---
layout: post
title: "空っぽのCentos7にコピペで作るMariaDB"
date:   2016-12-01 00:00:00 +0900
tags:
  - Linux
  - Centos7
published: false
---


* TOC
{:toc}

# はじめに

空っぽのCentos7にひたすらコピペで作るMariaDB。  
従来は MySQL だったけど、名前が変わったみたいね。  
たぶん、以下のやり方はMySQLでも使えるよ思うよ。  

# インストール

```
sudo su

yum -y install mariadb mariadb-server
yum -y install mariadb-devel

systemctl enable mariadb.service
systemctl start mariadb.service
systemctl status mariadb.service

```

# 初期化。

```
mysql_secure_installation
```

# 管理者ユーザの取り扱い

# テーブルの作成

# 作業ユーザの作成
