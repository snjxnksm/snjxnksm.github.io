---
layout: post
title: "Windows PackegeManagment"
date: "2017-03-12 15:41:11 +0900"
tags:
  - Windows
  - PackegeManagment
published: true
---

* TOC
{:toc}

# はじめに

Linuxユーザには、yumやapt-getなどといったコマンドに普段からお世話になっていると思う。   
なにかしらシステムを構築しようとした時、これらのパッケージ管理システムがないと、正直途方に暮れてしまう。  
パッケージ管理システムさえあれば、いちから環境を作成する際にも、インストーラをいちいち探さなくてもすむし楽チンである。  
ところが、わりと最近まで Windows なんてメジャーなOSに、これに相当するものがなかった。  
いや、非公式なものであれば chocoratey とかいくつかあるけど、公式なものはなかったはず。そこで出てきたのが、Windows10のPackegeManagmentだ。  

## 準備

PackegeManagmentはPoweShellで実行する。  
cortanaさんに聞いてみるか、コマンドラインを起動して、`> poweshell` と入力してみる。  
もし、初めて起動したなら、こんな感じで叱られるかもしれない。  

```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

. : このシステムではスクリプトの実行が無効になっているため、ファイル C:\Users\shinj_000\Documents\WindowsPowerShell\Mic
rosoft.PowerShell_profile.ps1 を読み込むことができません。詳細については、「about_Execution_Policies」(http://go.micros
oft.com/fwlink/?LinkID=135170) を参照してください。
発生場所 行:1 文字:3
+ . 'C:\Users\shinj_000\Documents\WindowsPowerShell\Microsoft.PowerShel ...
+   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   + CategoryInfo          : セキュリティ エラー: :slightly_smiling_face: ) []、PSSecurityException
   + FullyQualifiedErrorId : UnauthorizedAccess
```

起動時にこんな感じのエラーが出たら、スクリプトを使えるようにしておく。  
まずPwoeShellを管理者権限で起動して、以下のコマンドを実行する。  

```
set-executionpolicy remotesigned
```

次からは叱られない。  

## コマンド一覧

コマンド一覧は、microsoft のドキュメントを見て欲しい。  
[PackageManagement コマンドレット](https://msdn.microsoft.com/ja-jp/powershell/wmf/5.0/oneget_cmdlets)


## 現在インストールしているもの一覧

```
get-package
```

## インストールできるものを探す

```
find-package -name XXXX
```

## 新たにインストール

```
install-package -name XXXX
```

## アンインストール

```
Uninstall-Package -name XXXX
```

## chocorateyのリポジトリを使えるようにする。

```
Get-PackageProvider chocolatey
```

## 以上

以上！
