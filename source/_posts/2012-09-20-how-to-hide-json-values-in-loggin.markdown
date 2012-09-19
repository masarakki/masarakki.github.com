---
layout: post
title: "RailsログでJSONの一部を隠す方法"
date: 2012-09-20 05:10
comments: true
categories: rails
keywords: "rails,filter_parameters,log,config,しがらみ"
---

Rails のログ出力には filter_parameters という一部のパラメータを隠す便利な機能があります。

デフォルトでは config/application.rb に
```ruby
  config.filter_parameters += [:password]
```
と書かれています。

ここにフィルタしたいパラメータを追加すれば [FILTERED] という文字列に置き換えられます

例えば開発者が user_id を見てはいけない社内ルールがある場合
```ruby
  config.filter_parameters += [:password, :user_id]
```
とすればいいわけです。

## ためしてみる

### GETの場合

    GET /posts?password=abc123

を叩いた場合ログに

    Started GET "/posts?password=[FILTERED]" for 127.0.0.1 at 2012-09-20 05:16:44 +0900
    Processing by PostsController#index as HTML
      Parameters: {"password"=>"[FILTERED]"}

とURLもパラメータも [FILTERED] に変えられて出力されます。


### POSTの場合

    POST /posts
    data: password=abc123

を叩いた場合ログは

    Started POST "/posts" for 127.0.0.1 at 2012-09-20 05:18:12 +0900
    Processing by PostsController#create as */*
      Parameters: {"password"=>"[FILTERED]"}

上手いことフィルタされますね。

Rails の標準的なPOSTパラメータ形式の :post => {:password => "abc123"} はどうでしょうか

    Started POST "/posts" for 127.0.0.1 at 2012-09-20 05:22:33 +0900
    Processing by PostsController#create as */*
      Parameters: {"post"=>{"password"=>"[FILTERED]", "title"=>"unko"}}

これもきちんとフィルタできてます。

## 特殊な事情

ある特殊な事情から「パラメータは json というキーに json 文字列で入ってくる」というプロジェクトがある場合どうなるでしょうか?
ほんとにそんなプロジェクトがあるんでしょうか? ファックですね。
どうせ原因はこのAPIを叩いてくるphpが低脳だからに違いありません。

    POST "/post"
    data: json={\"password\":\"abc123\"}

という状況です。 ためしてみましょう。

    Started POST "/posts" for 127.0.0.1 at 2012-09-20 05:30:56 +0900
    Processing by PostsController#create as */*
      Parameters: {"json"=>"{\"user_id\":\"abc123\"}"}

残念、丸見えです! かと言って :json を全部フィルタするのは開発の妨げになってしまいます。

### 一番かっこ悪い解決方法

filter_parameters には、|key, value| を引数に取る lambda を指定することもできるようです。
サンプルでは
```ruby
lambda do |k,v|
  v.reverse! if k =~ %rsecret/
end
```
と書かれていますが、reverse! と破壊的メソッドを使っているのがすごく嫌な予感がします。
どうやら value のオブジェクトを破壊的に変更しないといけないようです。

こんな感じで書いてみました。

{% gist 3752166 %}

ログの出力結果は

    Started POST "/posts" for 127.0.0.1 at 2012-09-20 05:49:22 +0900
    Processing by PostsController#create as */*
      Parameters: {"json"=>"{:password=>\"[FILTERED]\"}"}

上手くいってます!!

ポイントは v.replace でオブジェクト自体は変えずに文字列を変更してる点です。
これを
```ruby
v = json.to_s
```
とやってしまうと上手く動きません。

とりあえずこれで泣く泣く json の入力すべてを捨てる必要はなくなりました。 やった!!

### 問題点

- [FILTERED] が定義されているクラスのファイルを require してもクラスが存在しないと言われる? ロード順の問題?
- 普通に指定されたフィルタしたいパラメータを lambda 内で再利用できない
  - lambda での指定は最後に評価されるので
  - before_filter (**まさに!**) があれば解決?

## ほんとにやりたい一番かっこいい解決方法

:json が渡ってきた場合は filter とかする前に JSON.parse してハッシュにしたい。
そうすれば普通に filter_parameters += [:password, :user_id] を指定するだけで、
:post => {:password => "abc123"} がフィルタできたのと同様にフィルタできるはず。

次はこれを調べる。
