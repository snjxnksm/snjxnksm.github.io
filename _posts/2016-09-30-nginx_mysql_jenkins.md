---
layout: post
title: Centos7でnginxとJenkinsサーバをたてる。
categories:
  - Linux
  - Centos7
  - nginx
  - Jenkins
published: false
---

* TOC
{:toc}

# はじめに

centos7にnginxを入れて、外部からアクセスできるようにする。  
その過程で、systemctlとfirewallの簡単な取り扱いを記述する。  
想定は、最低限のセットアップはされていて、SSH接続は可能であり、root権限を持っているサーバを対象とする。

## yumの準備

1. yum用ライブラリの追加  
    <pre>
    ※EPELを追加
    yum install epel-release -y
    </pre>  
2. update  
    <pre>
    yum update
    </pre>

## OSの設定  
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
    firewall-cmd --list-ports --zone=public
    </pre>

## nginx

0. nginx インストール

    <pre>
    yum install -y nginx
    </pre>

1. サービススタート  

    <pre>
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

### nginxの設定

/etc/nginx/nginx.conf がおおもとの設定ファイルだが、その中から /etc/nginx/conf.d/以下 をインクルードしている。  
なので、/etc/nginx/conf.d/に様々な条件のファイルを作成すればよい。  
インストール直後は以下の２ファイルがある。

* /etc/nginx/conf.d/default.conf  
    基本的なデフォルト設定が入っている。  
    変更した後は、以下のコマンドで文法が正しいか確認する。  
    <pre>
    nginx -t -c /etc/nginx/nginx.conf
    </pre>

* /etc/nginx/conf.d/example_ssl.conf  
    httpsでアクセスできるようにするための設定が入っている。  
    初期状態は中身がすべてコメントアウトされている。  
    証明書を入手したのちにコメントを外してからsystemctl restartをしてやればよい。  

#### 設定の例題
8080で稼働しているサービスを、httpsでつなげたい。

<pre>
[practice@test01 ~]$ cat /etc/nginx/conf.d/default.conf
server {
    listen       80;
    server_name  test.practice-test.com;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;


    location / {
        proxy_pass http://127.0.0.1:8089;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
</pre>

httpsでアクセスすると8080サーバが応答する。  
ただし、`https://FQDN/demoapp/` のみ、ローカルにある物理ファイルを対象とする。  

<pre>
[practice@test01 ~]$ cat /etc/nginx/conf.d/ssl.conf
# HTTPS server

server {
    listen       443 ssl;
    server_name  practice-test.com;

    ssl_certificate      /etc/nginx/ssl/practice-test.com_ServerCA_2016.crt;
    ssl_certificate_key  /etc/nginx/ssl/practice-test.com_2016.key;

    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers   on;

    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect                        off;
    }

    location /demoapp {
        alias   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
</pre>
