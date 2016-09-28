---
layout: post
title: Centos7でJenkinsサーバをたてる。
categories:
  - Linux
  - Centos7
  - Jenkins
---
# Jenkins君

ローカルPCにcentos7サーバを立てて、GUI環境を作る手順をメモする。  
ついでにjenkinsをインストールする手順もおまけ。  

* TOC
{:toc}


## 仮想機の用意

前提としてvirtualboxをインストールすること。  

* centos7  
https://www.centos.org/download/  
CentOS-7-x86_64-DVD-1511.iso  
（実行時の最新を利用すれば問題なし）  

* ポートフォワーディングルールを設定しておくこと  
外部から仮想機のjenkinsをたたくために以下の設定をする。  
virtualbox->Jenkins(VM)->設定->ネットワーク（アダプタ１）（高度）->ポートフォワーディング  

|名前|プロトコル|ホストIP|ホストポート|ゲストIP |ゲストポート|
|---|---|---|---|---|---|
|jenkins|TCP|(空白) |8081|(空白) |8081|
|ssh|TCP|(空白) |22|(空白) |22|



## 実行環境インストール  

1. GUI  
```
sudo yum -y groupinstall "Server with GUI"
sudo yum -y install alacarte
```  
2. 日本語環境  
```
localectl set-locale LANG=ja_JP.UTF-8
systemctl set-default graphical.target
```
3. Java  
```
yum install java-1.8.0-openjdk
yum install java-1.8.0-openjdk-devel
```
4. 再起動  
```
reboot
```
5. ユーザ作成  
shinjinakashima/shinjinakashimaで作成  
61. 実行環境インストール  
```
yum -y install epel-release
yum install -y bzip2 gcc make kernel-devel kernel-headers dkms gcc-c++
```
62. GuestAdditionsインストール  
    VMのメニュー→デバイス→GuestAdditionsCDイメージの挿入  
7. gitインストール
```
yum install git -y
```

## Jenkins インストール

以下の手順は、Jenkins単体で動作させる場合。

1. yumの準備  
jenkinsのリポジトリを登録する。
```
# wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
# rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
```
2. jenkinsをインストール
```
# sudo yum -y install jenkins
# chkconfig jenkins on
```
3. ポートの変更  
デフォルトだと8080なので、他のサービスとバッティングする場合は、以下を修正する。  
```
# vi /etc/sysconfig/jenkins
このなかの、JENKINS_PORTを 8080 から 8081 に変更する。
```
4. jenkins起動
```
# sudo systemctl start jenkins
```
6. 管理者ユーザの設定  
  まず管理者ユーザを作ったうえでanonymousでは何もできないようにしておく。  
  1. ログイン認証の追加  
    11. Jenkinsの管理＞グローバルセキュリティの設定  
      「セキュリティを有効化」にチェック  
    12. アクセス制御＞ユーザー情報＞「Jenkins のユーザーデータベース」、「ユーザーにサインアップを許可」にチェック  
    13. アクセス制御＞管理権限＞「ログイン済みユーザーに許可」にチェック  
    14. 保存 (トップ画面に戻される)  
  2. ユーザ作成  
    21. Jenkinsのトップ＞「Jenkinsの管理」＞「ユーザの管理」＞「ユーザ作成」  
    22. 「ユーザー名」、「パスワード」、「パスワードの確認」、「フルネーム」、「メールアドレス」を入力して、サインアップ  
  3. 権限付与  
    31. Jenkinsの管理＞グローバルセキュリティの設定  
    32. アクセス制御＞ユーザー情報＞「ユーザーにサインアップを許可」　のチェックオフ  
    33. アクセス制御＞ユーザー情報＞管理権限＞「行列による権限設定」 にチェック  
    34. 登録済みユーザー名(自分)を入力して、追加  
    35. ユーザー名(自分) の行のチェックを全てオン  
    36. 匿名ユーザー の行のチェックを全てオフ  
7. プラグインの登録  
  Jenkinsのトップ＞「Jenkinsの管理」＞「プラグインの管理」  
  * Gradle plugin 追加  
  * scm-api plugin 追加  
  * Git plugin 追加  
  * PegDown Formatter Plugin 追加  
  * Publish Over SSH Plugin 追加  


* windowsでやる場合は[bitnami版AllInOne](https://bitnami.com/stack/jenkins/installer#windows)などあります。
