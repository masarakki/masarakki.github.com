---
layout: post
title: "make url cool"
date: 2013-03-16 17:53
comments: true
categories:
---

## Railsでurlをカッコ良くする

railsで普通にこんな感じに config/route.rb を書くと、

```ruby
resources :genres do
  resources :books
end
```

urlはこんな感じになります。

```
 /genres/1/books
```

1ってなんだよ! 1って!!

## urlにidじゃないものをつかう

ジャンルのようなある程度数が決まっていて名前も変化しなさそうなのは、

```
 /genres/comic/books
```

みたいなurlにできるとカッコイイですよね。

genre_books_path(genre) のメソッドで、genre.idではなくgenre.nameにできれば上手くいきそうです。
このメソッドを調べると(追っていくのメチャメチャ難しかった)、
どうやら to_param というメソッドを経由して id が呼ばれていました。
このto_paramを書き換えてnameを返すようにしてしまっていいのでしょうか?

そこで [http://api.rubyonrails.org](http://api.rubyonrails.org) でto_paramを探すと、「ActionPackでurlを作るときに使うよ!」と、
しかもまさにやりたいことそのままのサンプルコードが置いてありました。

というわけで **to_param を書き換えてしまえばid以外にも自由にurlを作れる** ことがわかりました。

## to_param って・・・

url作るキーを取得するメソッドが to_param ってそんなの検索できねーよ・・・
名付けがクソだと人生の貴重な時間を無駄にするという例でした。

## 本当にやりたいこと

```
 /comics
 /novels
```

みたいなurlにしたい。 resourcesをきちんと使って。
