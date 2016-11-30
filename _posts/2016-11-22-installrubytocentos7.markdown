---
layout: post
title: 空っぽのCentos7にコピペで作るRuby環境。
categories:
  - Linux
  - Centos7
  - ruby
---

* TOC
{:toc}

# はじめに

空っぽのCentos7にひたすらコピペで作るrbenv+ruby-buildを使ったRubyの最新環境の作り方。  
とりあえず、rootのパスワードは確認すること。  
あとは、ひたすらコピペ。  
とにかくコピペ。  

# 環境作成
```
sudo su
yum groupinstall "Development Tools" -y
yum install epel-release -y
yum install -y openssl-devel readline-devel zlib-devel curl-devel libyaml-devel ImageMagick ImageMagick-devel
yum install nodejs -y
yum install sqlite-devel -y
```

# rbenv+ruby-buildインストール
```
cd /
git clone https://github.com/sstephenson/rbenv.git usr/local/src/rbenv
git clone https://github.com/sstephenson/ruby-build.git usr/local/src/rbenv/plugins/ruby-build

echo 'export RBENV_ROOT="/usr/local/src/rbenv/"' >> /etc/profile.d/rbenv.sh
echo 'export PATH="${RBENV_ROOT}/bin:${PATH}"' >>   /etc/profile.d/rbenv.sh
echo 'eval "$(rbenv init -)"' >>                    /etc/profile.d/rbenv.sh
cat /etc/profile.d/rbenv.sh

source /etc/profile.d/rbenv.sh
```

# ログインしたユーザでrbenvを使えるようにする。
```
exit
source /etc/profile.d/rbenv.sh
rbenv --version
```

# rubyのインストール
```
sudo su

rbenv install -l

rbenv install 2.3.1
rbenv global 2.3.1
```

# gem の準備
```
gem install rbenv-rehash
gem update --system
gem install rake
gem install bundler
gem install rails
```

# 以上

以上！
