---
layout: post
title: Centos7ではネットワークインターフェースをアクティブにするにはnmcliコマンドを使う。
categories:
  - Linux
  - Centos7
  - Centos6
---

* TOC
{:toc}

# はじめに

Centos7では、インストール時にネットワークを有効にしていないと、そのままネットワークインターフェースがアクティブになっていない状態になる。  
つまり、googleを見に行ったりもできないし、`yum update`等もできない。外部からsshログインなどもできない。  
意外にこれ、気が付かないと無意味にはまって時間ばかり食ったりする。  
これを、ちゃんと起動するようにする最小限の手順は以下の通り。  
もちろん、rootでやってね。

# nmcliコマンド

Cetnos7ではネットワーク管理用のコマンドが提供されている。  

<pre>
[root@localhost ~]# nmcli device
DEVICE    TYPE      STATE           CONNECTION
enp0s3    ethernet  disconnected    --
lo        loopback  unmanaged       --
[root@localhost ~]#
[root@localhost ~]#
[root@localhost ~]# nmcli con mod enp0s3 connection.autoconnect "yes"
[root@localhost ~]#
[root@localhost ~]# nmcli device
DEVICE    TYPE      STATE         CONNECTION
enp0s3    ethernet  connected     enp0s3
lo        loopback  unmanaged     --
[root@localhost ~]#
[root@localhost ~]#
</pre>

# ちなみにCentos6でも…

ちなみにcentos6でもminimalインストールとかすると、デフォルトではネットワークが有効になってはいない。  
centos6の場合は特別なコマンドがないので、サービスを止め、設定ファイルを編集して、サービス再開という手順になる。  

<pre>
[root@localhost ~]# service network stop
Shutting down loopback interface:          [ OK ]

[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
HWADDR=08:00:27:54:04:FC
TYPE=Ethernet
UUID=9aff0a74-bc1e-49a7-9992-bd7cc5eca9d3
ONBOOT=yes
NM_CONTROLLED=yes     # ←ここを no から yesに書き換える。
BOOTPROTO=dhcp

[root@localhost ~]# service network start
Bringing up loopbackInterface:             [ OK ]
Bringing up integerface eth0:
Determining IP information for eth0... done.
                                           [ OK ]
</pre>
