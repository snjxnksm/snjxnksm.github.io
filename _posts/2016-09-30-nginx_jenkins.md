---
layout: post
title: Centos7でnginxをたてて、Jenkinsを使えるようにする。
categories:
  - Linux
  - Centos7
  - nginx
  - Jenkins
---

* TOC
{:toc}

# はじめに

centos7にnginx、jenkinsを入れて外部からアクセスできるようにする。  

* その過程で、systemctlとfirewallの簡単な取り扱いを記述する。  
* jenkinsは、サブディレクトリで以下のようにアクセスできるようにする。  
  `http://example.com/jenkins`  
* nginxの設定で、サブディレクトリで受け取ったアクセスを、ポートフォワーディングで内部で動作しているjenkinsに引き渡すようにする。  

想定は、最低限のセットアップはされていて、SSH接続は可能であり、root権限を持っているサーバを対象とする。

# yumの準備

1. yum用ライブラリの追加  
    EPELを追加
    <pre>
    yum install epel-release -y
    </pre>  
2. update  
    <pre>
    yum update
    </pre>

# firewallのインストールと設定

firewallをインストールし、外部からのアクセスをポート80(http)に限定する。  

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
    firewall-cmd --add-port=80/tcp --zone=public --permanent
    firewall-cmd --reload
    firewall-cmd --list-all
    </pre>

# jenkinsインストールと設定

jenkinsのリポジトリを登録してインストールする。  

<pre>
# wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
# rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
</pre>
<pre>
# yum -y install jenkins
</pre>

##  設定の変更  

nginxと連携するための設定を記述する。  

<pre>
# vi /etc/sysconfig/jenkins
# このなかの、JENKINS_ARGSへ起動パラメータを追加する。
JENKINS_ARGS="--prefix=/jenkins --httpListenAddress=127.0.0.1"
</pre>

##  jenkins起動  

<pre>
# systemctl start jenkins
</pre>

##  管理者ユーザの設定  

ここまでの作業で、8080ポートにてjenkinsが動作しているはず。  
もし、作業しているCentOS7がGUIインストールされている場合は、バンドルされているfirefoxなどのブラウザで `http://localhost:8080/jenkins` にログインすると、権限を設定せよという画面出てくる。  
Administrator Password欄に、画面に指示されているファイルの中身を入力する。  
そのあとは初期設定を実行する。
初期設定画面では、プラグインの指定と全権管理者を設定する。


# nginx

0. nginx インストール

    <pre>
    yum install -y nginx
    </pre>

1. サービススタート  

    <pre>
    systemctl enable nginx
    systemctl start nginx
    </pre>

2. ステータス確認

    <pre>
    systemctl status nginx
    </pre>

3. サービス停止
    <pre>
    systemctl stop nginx
    </pre>

## nginxの設定

「location」でサブディレクトリを指定して、そのさきにjenkinsをつなげる。  
/etc/nginx/nginx.conf に以下を追加。

<pre>
[root@localhost log]# cat /etc/nginx/nginx.conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        ##  Jenkins向け設定 ここから
        location /jenkins {
            proxy_pass  http://localhost:8080/jenkins;
        }
        ## ここまで

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

}

</pre>
