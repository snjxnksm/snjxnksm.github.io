---
layout: post
title: Centos7ではネットワークインターフェースをアクティブにするにはnmcliコマンドを使う。
categories:
  - Linux
  - Centos7
---

* TOC
{:toc}

# はじめに

Centos7では、インストール直後はネットワークインターフェースがアクティブになっていない。  
つまり、googleを見に行ったりもできないし、`yum update`等もできない。  
これを、ちゃんと起動するようにする最小限の手順は以下の通り。  

# nmcliコマンド

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
