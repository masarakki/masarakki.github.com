---
layout: post
title: "run ruby on supervisor with RVM"
date: 2013-02-14 16:17
comments: true
categories: ruby
keywords: ruby,rvm,supervisor
---

## supervisorとは

supervisorとは、pythonで書かれたプロセス監視ツールです。
簡単にインストールでき、デーモンとして動作し、
マシンブート時に任意のプログラムを立ち上げたり、そのプロセスが死んだ時に自動再起動などができます。

動かすプログラムは、単純な無限ループするようなコードでよく、
プロセス永続化などの面倒な仕事は全部supervisorに任せてしまえます。

## RVMで使う時の問題点

RVMは個人環境にrubyをインストールし、実行するためには環境変数の整備などが必要で、
通常はシェルの起動時に設定スクリプトを読み込んで上手いことやってくれます。
ですが、supervisorを使う場合その方法が取れないのでなんとかしないといけません。

supervisorはenvironmentで環境変数を設定できるのですが、

```
$ rvm env
```

で出力した環境変数を使っても上手くいきませんでした。

```
command=which bundle
```

は想定したpathを返すのですが、

```
command=bundle exec ...
```

にすると bundle not found. と・・・

```
command=cd /path/to/project ; bundle exec ...
```

にしてみても、上手いこと解釈してくれないし、そもそもcdを見つけてくれません。 (シェル組み込みコマンドだから?)

もう意味わかりません。


## そう言えば昔wrapperってあったよね

そういえば昔まだbundlerがなくてpassengerが主流だったときは、
rvm wrapper とかいうのやってそれを使えって書いてあるサイトが多かったと思います。
なんかマルっと環境を上手いことやってくれるものだっていう認識でした。
試してみましょう。

なんか適当に名前を付けてwrapperを作ります。
gemsetも一緒に hoge_task とかの名前にします。

```
$ rvm wrapper 1.9.3-p385@hoge_task hoge_task bundle
```

これで $HOME/.rvm/bin に hoge_task_bundle というコマンドができあがりました。
実際にフルパスでこのコマンドを叩くと hoge_task のgemsetを使ってbundleが動いているのがわかります。
supervisorのcommandの設定にこれを使えば良さそうです。

supervisorの設定ファイルは全体でこんな感じです

### /etc/supervisor/conf.d/hoge_task.conf

```
[program:hoge_task]
command=/home/{USERNAME}/.rvm/bin/hoge_task_bundle exec rackup -p 8000
autostart=true
autorestart=true
stopsignal=QUIT
stdout_logfile=/var/log/hoge_task/out.log
stderr_logfile=/var/log/hoge_task/err.log
user={USERNAME}
diretory=/home/{USERNAME}/projects/hoge_task
```

あとは

```
$ sudo supervisorctl reload
$ sudo supervisorctl status
```

とかやって設定を読み込み直し、結果を見てみましょう。
多分うまく動き出してるはずですよ!

## Gemfileやruby自体の更新

プログラムを変更してGemfileが変わった時や、
rubyのバージョンアップをした時などはどうなるのでしょうか?

これもGemfileのアップデートなら

```
$ bundle install
```

で、いつもどおりgemをインストールした後、(もちろん @hoge_task のgemsetを使って!)

```
$ sudo supervisorctl restart hoge_task
```

でプロセスを再起動すればいいはずです。

rubyを更新した場合は、新しいバージョンを指定してrvm wrapperを作りなおせば、
hoge_task_bundle の中身が書き換えられるので、supervisorの設定はそのままで大丈夫です。
あとは上と同様に bundle install してプロセスの再起動を指示します。

## もう何も怖くない

これはsupervisorの設定と言うよりむしろ、
ユーザ環境にインストールされたrvmをシステムに近いところで使う汎用的な方法のようです。
自動でrackアプリを起動したり、バックグラウンドでタスクを回したり、いろいろできるようになりました。

というわけで仕事で使うメモでした。
