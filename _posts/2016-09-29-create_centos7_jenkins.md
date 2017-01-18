---
layout: post
title: Centos7でJenkinsサーバをたてる。
categories:
  - virtualbox
  - Linux
  - Centos7
  - Jenkins
---

* TOC
{:toc}

# はじめに

ローカルPCにcentos7サーバを立てて、Jenkinsをインストールするまでの手順を書く。  

# 仮想マシンの用意

割と有名なvirtualboxを使ってみる。  

## virtualboxをインストール  

https://www.virtualbox.org/

[ダウンロードページ](https://www.virtualbox.org/wiki/Downloads)から各プラットフォームにあったものを選んでインストールする。

## centos7を取得  

https://www.centos.org/download/  
*DVD ISO* をクリックすると、最小限セットのisoファイルが手に入る。  
通常はこれをDVDなどに焼き付けるが、今回はこのまま使用する。  
（実行時の最新を利用すれば問題なし）  
適当なフォルダにダウンロードしておく。  

## 仮想マシン作成

1. virtualboxを起動し、「新規(n)」をクリック  
2. 仮想マシンを作成する。
    21. 適当な名前をつける。  
    22. タイプを「Linux」にする
    23. バージョンを「Red Hat (64-bit)」にする  
    24. メモリサイズは、1024くらいにしておく（あとで変更可能）
    25. 「仮想ハードディスクを作成する」を選択する  
    26. ハードディスクの設定を行う
        261. ファイルサイズは、ホストマシンの空き容量をみて、適当に。  
          30GBくらいでいいんじゃないですか？  
          ただし、ハードディスクのサイズをあとで変更しようとするとかなり面倒なので、適当かつ慎重に。
        262. ファイルタイプは VDI が推奨。
        263. ストレージは「可変サイズ」を選択。
3. Centos7のDVDイメージを仮想マシンに挿入する。
    31. 新たに作成され、「電源オフ」状態の仮想マシンを選択する。  
    32. 「設定(S)」をクリック  
        321. ストレージをクリック  
            3211. 「空」となっている光学ドライブを選択  
            3212. 「属性」の部分の光学媒体のアイコンをクリックしてアイコンをクリックしてファイルを選択  
            3213. ダウンロードして来たCentOS-7-x86_64-DVD-1511.isoを選択する。  
4. 仮想マシンを起動する。  
  「起動(T)」クリック。  
5. Centos7インストーラを設定する。  
  最低限必要なのは、「インストール先(D)」の設定。  
  一度、「インストール先」画面に移動してデフォルトのディスクにチェックが立っていることを確認して「完了」をクリックする。  
  作りたい環境に合わせて、「ソフトウェアの選択」で環境を作成する。  
  あえていばらの道を行きたければ、デフォルトである「最小限のインストール」でOK。ただし、この設定だとホントにコマンドラインしかインストールされない。  ß
  そうでもない普通の人は、「サーバ(GUI使用)」に「開発ツール」アドオンをつけた構成がおすすめ。  
    51. インストール中に root パスワードは入れておくこと  
    52. 管理ユーザの作成も実行しておく。  
6. インストールが終了したら「再起動(R)」  
7. 再起動したら　License の同意をする。  
8. 再起動した直後は *ネットワークがオフ* になっている。  
    まずはネットワークを接続すること。GUIをインストールしたら設定画面から辿れる。  

# 実行環境インストール  

1. GUI  
    以下は、Centos7インストーラで「サーバ(GUI使用)」を選択した場合と同等の作業。  
    結構時間がかかる。  
    <pre>
    sudo yum -y groupinstall "Server with GUI"
    sudo yum -y install alacarte
    </pre>
2.  Java  
    以下は、Centos7インストーラで「Javaプラットフォーム」アドオンを選択した場合と同等の作業。
    jenkinsを走らせるのに必要。
    <pre>
    yum -y install java-1.8.0-openjdk
    yum -y install java-1.8.0-openjdk-devel
    </pre>
3.  開発環境インストール  
    以下は、後述のGuestAdditionsをインストールする際に必要になる。  
    <pre>
    yum -y update kernel    
    yum -y install epel-release
    yum install -y bzip2 gcc make kernel-devel kernel-headers dkms gcc-c++
    </pre>
4.  日本語環境  
    <pre>
    localectl set-locale LANG=ja_JP.UTF-8
    systemctl set-default graphical.target
    </pre>
5.  gitインストール  
    <pre>
    yum install git -y
    </pre>
7. yum updateしておく
    結構時間がかかるので覚悟してやる。  
    <pre>
    yum -y update
    </pre>
7.  再起動  
    <pre>
    reboot
    </pre>

##  GuestAdditionsインストール  
マウスカーソルのシームレスな移動や、ホストとゲストの間のクリップボード経由のコピペなどをやりたい場合は、GuestAdditionsCDを仮想マシンにインストールする。  
* VMのメニュー→デバイス→GuestAdditionsCDイメージの挿入  
* 実行するかどうか聞かれるので実行する。
* 無事に実行できたら、再起動

# Jenkins インストール

以下の手順は、Jenkins単体で動作させる場合。

##  yumの準備  
jenkinsのリポジトリを登録する。  
rootで実行すること。  

<pre>
# wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
# rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
</pre>

##  jenkinsをインストール  

<pre>
# yum -y install jenkins
</pre>

##  jenkins起動  

<pre>
# systemctl start jenkins
</pre>

##  管理者ユーザの設定  

ここまでの作業で、8080ポートにてjenkinsが動作しているはず。  
バンドルされているfirefoxなどのブラウザで http://localhost:8080 にログインすると、権限を設定せよという画面出てくる。  
Administrator Password欄に、画面に指示されているファイルの中身を入力する。  
そのあとは初期設定を実行する。
初期設定画面では、プラグインの指定と全権管理者を設定する。

##  プラグインの登録  
Jenkinsのトップ＞「Jenkinsの管理」＞「プラグインの管理」  
* Gradle plugin 追加  
* scm-api plugin 追加  
* Git plugin 追加  
* PegDown Formatter Plugin 追加  
* Publish Over SSH Plugin 追加  



# 外部のマシンのブラウザで仮想マシンのJenkinsを覗く設定  

以下の設定を施すと、外部から仮想マシンの8080ポートを覗けるようになる。

## 仮想マシンにfirewallをインストールする。

0. firewall をインストール
    <pre>
    yum install -y firewall
    </pre>
1. firewall を有効化  
    <pre>
    systemctl enable firewalld
    systemctl start firewalld
    </pre>
2. ポートを開ける  
    <pre>
    firewall-cmd --add-port=8080/tcp --zone=public --permanent
    firewall-cmd --reload
    firewall-cmd --list-ports --zone=public
    </pre>

## ポートフォワーディングの設定をする。  

virtualbox->Jenkins(VM)->設定->ネットワーク（アダプタ１）（高度）->ポートフォワーディング  

|名前|プロトコル|ホストIP|ホストポート|ゲストIP |ゲストポート|
|---|---|---|---|---|---|
|jenkins|TCP|(空白) |8080|(空白) |8080|
