---
layout: post
title: "remove uploaded files in Rails (with unicorn)"
date: 2012-09-15 15:19
comments: true
categories: [ruby,rails,unicorn]
description: "rails + unicorn でアップロードされたファイルを消す方法"
keywords: "rails,unicorn,uploade,file,clean,remove,phpdis"
---

Rails + unicorn環境でファイルアップロードを扱うと /tmp/RackMultipart... というファイルが作られなかなか消えません。
UploadedFileクラスの中身はTempfileなので
```ruby
params[:file].close(true)
```
とかやってみてもやっぱり消えません。
どうやらunicornを再起動して少し経つとGCが起動して消えるような動作をします。

これに対処するため某社では5分に一度unicornを再起動しているようです。
これではまるで「僕なら、10リクエストごとにApacheを再起動しますね」と言ってるバカと一緒じゃないですか。
そんなのはよくないですよね。

というわけで色々やってみた結果 「すぐには消えないけど高々プロセス数以下のファイルしか残らない方法」 を見つけました

```ruby
def remove_uploaded(file)
  tmp_path = file.instance_variable_get(:@tempfile).path
  File.unlink(tmp_path) if File.exists?(tmp_path)
  file.instance_variable_set(:@tempfile, nil)
end
```

このままッ!! nilを! こいつの! @tempfileに......... つっこんで! 殴り抜けるッ!

これで/tmpが無限に肥大化することはなくなるようです
