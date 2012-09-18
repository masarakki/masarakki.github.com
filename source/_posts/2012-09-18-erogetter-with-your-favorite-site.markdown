---
layout: post
title: "EroGetter with your favorite site!"
date: 2012-09-18 08:06
comments: true
categories: [ruby, ero]
description: EroGetter対応サイトを増やして快適エロ生活
keywords: "ruby,gem,ero,ero_getter"
---

## EroGetter対応サイトを増やす

サイト定義を書くことで対応サイトを増やすことができます。

### 準備するもの
- まともな(ry
- 対象サイト
  - 記事urlの正規表現
  - サイト名
- エロにかける情熱

今回追加するサイトは **http://example.com/archives/\d.html えろいぐざんぷるどっとこむ** とします。

#### ダウンロード


    git clone git://github.com/masarakki/ero_getter.git
    cd ero_getter
    bundle install
    git checkout -b add_example_dot_com


#### クラス名の決定
サイト名が **えろいぐざんぷるどっとこむ** なのでクラス名は **EroExample** とします。
通例に従って、ファイル名などは **ero_example** のようなスネークケースになります。

#### サンプルファイルの追加

    mkdir spec/samples/ero_example
    wget http://example.com/archives/1234567.html -o spec/samples/ero_example/sample.html

その1〜 のように連番になっているサイトの場合は先頭、中間、最後のファイルをそれぞれ別名で保存してください。

#### テストの追加
*spec/downloader/* にある他のサイト定義のファイルをコピーして、
**spec/downloader/ero_example_spec.rb** を作ります。
連番サイトは *nijigazou_sokuhou_spec.rb*、 そうでないサイトはそれ以外のファイルがいいでしょう。

ファイルを開いてそれっぽいところを変更します。

```ruby
describe EroExample do                                     # クラス名を変更
  subject { @dl }
  let(:url) { 'http://example.com/archives/1234567.html' } # urlを変更
  before do
    FileUtils.stub(:mkdir_p)
    @dl = EroExample.new(url)                              # クラスを変更
    fake(:get, url, 'ero_example/sample.html')             # サンプルファイルを変更
  end
  its(:sub_directory) { should == '1234567' }              # サブディレクトリ名を変更
  its("targets.count") { should == 26 }                    # サンプルファイルの画像数に変更
end
```

    rspec spec/downloader/ero_example_spec.rb

を実行してFailがひとつも出なくなるようにサイト定義をいじります。

別のターミナルを立てて

    bundle exec guard start

でguardを動かしてCIすることもできます。

#### サイト定義の追加

**lib/downloader/ero_example.rb** に、 **EroGetter::Base** を継承したクラスを作ります。
対象サイトがLivedoorBlogの場合は、 **EroGetter::Livedoor** が使える場合もあります。

```ruby
class EroExample < EroGetter::Base
  name 'えろいぐざんぷるどっとこむ'
  url %r{http://example.com/archives/\d+.html}
  target "a > img.ero-file" do |tag|
     tag.parent[:href] if path.parent[:href] =~ /jpe?g|gif|png/
  end
  sub_directory do
     url.match(/(\d).html/)[1]
  end
  filename {|attr| "%04d%s" % [attr[:index], attr[:ext]] }
end
```

肝になるのは **target** のところです。
targetで取得したい画像のurlを集める方法を指定します。

targetの引数には取得したいタグをcssで指定します。
そのタグに対してブロックの処理をしてurlを取得します。
最終的に、 **画像のurlの文字列** を集めなければならないことに注意してください。
(ただタグを返すだけだとurl文字列ではないということです)

この例の場合、aタグ直下の、クラスにero-fileが付けられたimgタグを全部持ってきて、
その親要素のhref属性が画像ファイルなら、そのhrefを文字列として返す、という条件になります

rspecを走らせて、取得した画像の件数が一致するまでtargetを調整しましょう。

#### 連番サイト
その1〜 など連番で記事が作られるサイトは、ひとつの記事から全てのページを取得する設定が可能です。

*lib/downloader/nijigazou_sokuhou.rb* を参考にしましょう。

```ruby
  connection ['a[rel=prev]', 'a[rel=next]'] do |path|
    path.text.match(Regexp.escape(title_part))
  end

  def title_part
     title.match(/(.+?)(その.+)?$/)[1]
  end
```

**connection** で前後のページへのリンクをcssの配列で指定します。
ブロックを渡してtrue/falseを返すことで、そのリンクを本当に取得するかどうかを調整できます。
この場合、タイトルの「その〜」より前の部分が一致したら本当に取得する事になります。

rspecも *spec/downloader/nijigazou_sokuhou_spec.rb* を参考に、
前後ページが正しく取得できるか、次がなければnilになるか、をテストしましょう。

#### Pull Request

あとはPull Requestすれば取り込まれます。
Pull Requestの前にはコミットをひとつにまとめるとかっこいいですよ。

    git rebase -i HEAD~10 (十分大きい数字)

コミットIDの前の文字列を **s** にすると、その前のコミットと合成されます。
作業をひとつのコミットにまとめて、コミットメッセージを適切なものに直し、
pushしてPull Requestで完成です!

*Enjoy!*
