---
layout: post
title: "Gitのブランチ戦略"
date: "2017-07-09 14:40:45 +0900"
tags:
  - Git
  - 構成管理
published: false
---

* TOC
{:toc}

# Gitブランチ戦略

Gitを、だね。使うわけだよ。  
でも、どんな使い方がるあるの？って探した時に、よく似た名前で微妙に違う説明が書いてあって、混乱してしまう。  
で、以下にざっと目についたブランチ戦略をリストアップしてみた。  

| 名前 | 特徴 |
|----|----|
| Git Flow | 一番メジャー</br>Gitコマンドのプラグインとして作られていて、専用のサブコマンドが用意されている<br>SourceTree でGUI操作が可能。<br>運用するブランチが少し多い。<br>ブランチandマージ方式。  |
| GitHub Flow | 手動で実施する。<br>運用するブランチは少ない（といいつつtopicブランチの下は多くなる）。<br>開発者ごとにリポジトリを保有する。<br>フォークandプルリク方式。 |
| Gitlab Flow | 手動で実施する。<br>運用するブランチは少し多い。<br>リリース対象のブランチを用意する。<br>全員でひとつのリポジトリを参照する。<br>ブランチandマージ方式。 |

## Git Flow

おそらくGitを使ったワークフローを探すと一番最初に行き当たる手法。  
運用管理するブランチが少し多いが、Gitのプラグインなどが提供されており、手順の複雑な部分は隠蔽されている。SourceTreeにGUIのインターフェースもある。  
例えば、featureブランチは必ずmasterから分岐し、作業が終わればmasterへマージするなどのルールをプラグインが保証してくれる。  
機能追加の最中に緊急でバグ対応を迫られた場合などの手順が用意されている。  

| ブランチ名称 | 機能 |
|----|---|
| master | すべての基本<br>開発者は基本的にここにはコミットしない<br>いつ何時でも、常にリリースできるような状態を保つ。<br>最後まで生き残る。|
| develop | 作業ブランチ<br>機能追加はここから分岐し、いったんここへ合流する。<br>最後まで生き残る。 |
| feature | 機能追加<br>developより分岐し、developへ戻る。<br>作業が済んだらブランチそのものは削除する。|
| release | リリース準備<br>全ての機能追加作業が済んだら（つまりdevelopから分岐したfeatureが全てマージされて合流したら）、developから分岐する。<br>リリースが無事に完了したら、developとmasterの両方にマージすることで、masterとdevelopの同期を取る。<br>作業が終わったらこのブランチも削除。  |
| hotfix | 緊急のバグフィックス<br>これだけはmasterから分岐する。<br>不具合対応が済んだら、これもdevelopとmasterの両方にマージすることで、それぞれのブランチの同期を取る。<br>作業が終わったらこのブランチも削除。  |

### 参考URL

* [git-flowについて](https://gist.github.com/Getaji/f5fa9b588bf1bfa6e21a) （コマンドラインでの使い方概略）  
* [Git-flowって何？ Qiita](http://qiita.com/KosukeSone/items/514dd24828b485c69a05)
* [SourceTree上でGit Flowを動かしてみる Qiita](http://qiita.com/masatomix/items/5e520591695f21769f11)
* [git-flowを使った日々の開発フローのふりかえり Qiita](http://qiita.com/y_minowa/items/430439448943b21dbff6)  
* [git flow × Pull Request を使ってモダンな開発してみた Qiita](http://qiita.com/Tamiiy/items/86f122d40ef6b158c2ab)
* [git-flowを使ってGitHubにプルリクエストを送ってみる手順 Qiita](http://qiita.com/ycoda/items/7faf1863b98eb584daf6)

## GitHub Flow

とくにプラグインやGUIはない。手動で実施する。  
フォーク先からプルリクを送り、それを受け入れることでレビューが完了とする。  
masterブランチは、つねにリリースできるようにする。  
(…ことを人間が気を付ける)  

| ブランチ名称  | 機能 |
|---|----|
|master       | すべての基本<br>つねにリリースできるようにする。  |
|topic|トピックブランチ<br>機能追加・不具合対応に使う。<br>topic/task_name1<br>topic/task_name2<br>などのように、複数のブランチを持つ。 |

### 参考URL

* [GitHub Flow 日本語訳](https://gist.github.com/Gab-km/3705015)

## GitLab Flow

とくにプラグインやGUIはない。手動で実施する。  
関係者全員は同じリポジトリを見る。フォークはしない。  
リリース用のブランチを用意する。  
GitHubではなく、[Gitlab](https://about.gitlab.com/) が元となる。  
Gitlabでは、ブランチでの作業が終わったらマージリクエストを発行する。  
(Githubでも同様の機能はある模様)  

| ブランチ名称 | 機能 |
|---|---|
| master      | すべての基本
| topic       | 機能追加・不具合対応に使う。 |
| production  | リリースブランチ |

### 参考URL

* [アプリ開発にはGitlab flowが合うと思います](http://shoma2da.hatenablog.com/entry/2015/11/04/233534)  
* [GitLab flowから学ぶワークフローの実践](http://postd.cc/gitlab-flow/)
* [Pull Request / Merge Request の違い Qiita](http://qiita.com/pink/items/8ab3ecc270a9a7db46b4)
* [GitHub初心者はForkしない方のPull Requestから入門しよう](http://blog.qnyp.com/2013/05/28/pull-request-for-github-beginners/) フォークをしないプルリク +
