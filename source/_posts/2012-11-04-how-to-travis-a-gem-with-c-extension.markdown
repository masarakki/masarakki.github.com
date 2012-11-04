---
layout: post
title: "Travis CIでrubyのC拡張が入ったgemのテストをする方法"
date: 2012-11-04 17:11
comments: true
categories: ruby
keywords: "ruby,travis,c"
description: "C拡張が入ったgemはどこにコンパイルされるのという話"
---

## No Such File

C拡張が含まれたgemをそのままTravisに流してみると、

    $ bundle exec rake
    56/home/vagrant/.rvm/rubies/ruby-1.8.7-p358/bin/ruby -S rspec spec/jpeg_spec.rb
    57/home/vagrant/builds/masarakki/jpeg/spec/../lib/jpeg.rb:2:
      in `require': no such file to load -- jpeg.so (LoadError)

自分でコンパイルしないといけないみたいですね。

## とりあえずやってみた

試しに.travis.ymlでコンパイルの命令を追加してみました。

```yaml
before_script:
- cd ext ; ruby extconf.rb ; make install
```

やった! Travisがテスト全部通したよ!!!

Travisの実行ログを見てみると、

    55$ cd ext ; ruby extconf.rb ; make install
    56checking for jpeglib.h... yes
    57checking for main() in -ljpeg... yes
    58creating Makefile
    59gcc -I. -I. -I/home/vagrant/.rvm/rubies/ruby-1.8.7-p358/lib/ruby/1.8/i686-linux -I. -DHAVE_JPEGLIB_H  -D_FILE_OFFSET_BITS=64  -fPIC -g -O2  -fPIC   -c jpeg.c
    60gcc -I. -I. -I/home/vagrant/.rvm/rubies/ruby-1.8.7-p358/lib/ruby/1.8/i686-linux -I. -DHAVE_JPEGLIB_H  -D_FILE_OFFSET_BITS=64  -fPIC -g -O2  -fPIC   -c jpeg_error.c
    61gcc -I. -I. -I/home/vagrant/.rvm/rubies/ruby-1.8.7-p358/lib/ruby/1.8/i686-linux -I. -DHAVE_JPEGLIB_H  -D_FILE_OFFSET_BITS=64  -fPIC -g -O2  -fPIC   -c jpeg_jpeg.c
    62gcc -shared -o jpeg.so jpeg.o jpeg_error.o jpeg_jpeg.o -L. -L/home/vagrant/.rvm/rubies/ruby-1.8.7-p358/lib -Wl,-R/home/vagrant/.rvm/rubies/ruby-1.8.7-p358/lib -L.  -rdynamic -Wl,-export-dynamic    -Wl,-R -Wl,/home/vagrant/.rvm/rubies/ruby-1.8.7-p358/lib -L/home/vagrant/.rvm/rubies/ruby-1.8.7-p358/lib -lruby -ljpeg  -lrt -ldl -lcrypt -lm   -lc
    63/usr/bin/install -c -m 0755 jpeg.so /home/vagrant/.rvm/rubies/ruby-1.8.7-p358/lib/ruby/site_ruby/1.8/i686-linux



        ＿人人人人人人人人人＿
        ＞　site_ruby の下　＜
        ￣^Y^Y^Y^Y^Y^Y^Y^￣

これはダメですねヤバイですねやっちゃダメですね。

## rake-compiler を使う

[rake-compiler](https://github.com/luislavena/rake-compiler) というgemを使うと上手いことやってくれるようです。

#### gem追加

Gemfileに以下を追加

```ruby
gem 'rake-compiler'
```

んで

    $ bundle install

#### タスク追加

Rakefileに以下を追加

```ruby
if RUBY_PLATFORM =~ /java/
  require 'rake/javaextensiontask'
  Rake::JavaExtensionTask.new('GEM_NAME')
else
  require 'rake/extensiontask'
  Rake::ExtensionTask.new('GEM_NAME')
end
```

#### rake-compiler流にディレクトリ構成を直す

ext直下にextconf.rbがある場合は、ext/GEM_NAME/extconf.rb の形になるようにディレクトリ構成を変更します。

#### ためしてみる
rake compile してみる

    $ rake compile
    mkdir -p tmp/x86_64-linux/jpeg/1.9.3
    cd tmp/x86_64-linux/jpeg/1.9.3
    /home/masaki/.rvm/rubies/ruby-1.9.3-p194/bin/ruby -I. ../../../../ext/jpeg/extconf.rb
    checking for jpeglib.h... yes
    checking for main() in -ljpeg... yes
    creating Makefile
    cd -
    cd tmp/x86_64-linux/jpeg/1.9.3
    make
    compiling ../../../../ext/jpeg/jpeg_jpeg.c
    ../../../../ext/jpeg/jpeg_jpeg.c: In function ‘jpeg_jpeg_exit’:
    ../../../../ext/jpeg/jpeg_jpeg.c:32:5: warning: format not a string literal and no format arguments [-Wformat-security]
    compiling ../../../../ext/jpeg/jpeg.c
    compiling ../../../../ext/jpeg/jpeg_error.c
    linking shared-object jpeg.so
    cd -
    install -c tmp/x86_64-linux/jpeg/1.9.3/jpeg.so lib/jpeg.so

環境が汚れなさそうなのは確認できましたね。

*warning直さないと・・・*

#### travisに教える
.travis.yml に以下を追加 (もちろん前のは消して)

```yaml
before_script:
 - rake compile
```

#### travisのログ確認

    39$ rake compile
    40mkdir -p tmp/i686-linux/jpeg/1.8.7
    (中略)
    54install -c tmp/i686-linux/jpeg/1.8.7/jpeg.so lib/jpeg.so

環境も汚してないしテストも通りました! やったね!

## 参考
- [Travisの実行結果](https://travis-ci.org/#!/masarakki/jpeg)
- [テスト対象プロジェクト](https://github.com/masarakki/jpeg)
