---
layout: post
title: "ヘルパーメソッドのspecってどう書くの?"
date: 2012-09-10 11:24
comments: true
description: "ヘルパーのrspecの書き方がわからなくて困っているという話"
keywords: "ruby,rails,helper,rspec"
categories: [rails, rspec]
---

[twitter-bootstrapのよくあるパターンを簡単にかけるヘルパーを集めたgem](https://github.com/masarakki/simple_bootstrap_helpers) を作ろうとしてみた
けどrspecの上手い書き方がわからない

例えばフォームの実行ボタン類を置くエリアを作るヘルパー

#### helper

```ruby
require 'action_view'
module BootstrapHelpers
  include ActionView::Helpers::TagHelper
  def actions(&block)
    content_tag(:div, :class => 'form-actions', &block)
  end
end
ActionView::Helpers.send(:include, BootstrapHelpers)
```

##### 使い方(haml)

```haml
= actions do
  f.button :submit, :class => "btn btn-primary"
```

一応これで使えるには使えるが
もちろんこれのrspecを書きたい

#### helper_spec.rb
```ruby
describe BootstrapHelpers
  include BootstrapHelpers

  it "div.form-actionsで囲まれる" do
     tag = actions do
        'hoge'
     end
     tag.should match /class="form-actions"/
  end
end
```

みたいな感じで書けると思ったけど
実行してみると
```ruby
undefined method `output_buffer=' for #<RSpec::Core::ExampleGroup::Nested_1::Nested_1:0x0000000241e5b8>
```
どうやらblockを渡すと capture(&block) される中でoutput_buffer=ねーよって言われるらしい
となるとblock渡すヘルパーはことごとくspecが書けないことになる

これ以上 helper_spec に余計なこと書かずにこのspecを実行できるようにするにはどうすればいいのだろうか?
(include BootstrapHelpers の1行だけでも十分余計である)

それともそもそも書き方の方針が大外れしてんのかねぇ
