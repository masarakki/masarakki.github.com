---
layout: post
title: "how to use EroGetter"
date: 2012-09-16 18:12
comments: true
categories: [ruby,ero]
description: EroGetterで快適エロ生活
keywords: "ruby,gem,ero,ero_getter"
---
*快適エロ生活はEroGetterで。*

## EroGetterとは?

エロ画像収集に特化したダウンローダーです。
汎用ダウンローダーを使う場合に必要になる、余計なURLを除外したり、保存場所を指定したりといった面倒な部分を極力排除してあります。
サーバとしても動作するので自宅サーバに立てておけば会社にいながら今夜のオカズを集めておくこともできます。

サイト定義を追加してプルリクエストすることにより対応サイトを増やすことができます。

## 使い方

収集だけが目的の場合はこれだけでokです

### 用意するもの

- まともにRubyが動くまともなOSのコンピュータ
  - つまりWindowsでの動作報告を待っています
- ruby-1.9.3のインストール
- Google Chrome / Chromium

#### Chromeエクステンションのインストール

```
git clone git://github.com/masarakki/ero_getter_chrome_extension.git
cd ero_getter_chrome_extension
bundle install
#( src/ero_getter.coffee の url: を変更してero_getter_serverのホストを変更可能 )
make
```

Chromeの拡張ページで ero_getter_chrome_extension のディレクトリを指定してインストール。
Chromeのアドレスバーの右側におっぱいアイコンが追加されたら成功です。

#### サーバのダウンロード

```
git clone git://github.com/masarakki/ero_getter_server.git
cd ero_getter_server
bundle install
```

#### バックグラウンドタスクの実行

```
rake backend:start
```

#### サーバの起動

```
rackup -p 9393
```

[http://localhost:9393/](http://localhost:9393/) にアクセスするとサーバステータス、キュー、対象サイトのリストなどが見れます。

あとは対象画像サイトの記事ページでおっぱいアイコンをクリックすれば $HOME/ero_getter 以下に保存されます


### 更新

```
git pull origin master
```

更新があったら
```
bundle install
rake backend:restart
ps ax | grep rackup で プロセス探して kill
rackup -p 9393
```

とりあえず今回は使い方だけのまとめで終了ー
