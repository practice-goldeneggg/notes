## 逆引き

### シンボル
* [Railsを触る際知っていると便利なRubyの基礎 [ブロックとかシンボルとか] - Qiita](http://qiita.com/kidachi_/items/46a6e49b6306655ccd64#2-6)
    * 文字列と近いが、少々性質が異なる。
    * 内部的な比較の際、文字列は1文字ずつ比べる必要があるが、シンボルは一度に全体を比較可能。
        * => ハッシュ等のキーとして理想的な性質。
    * 文字列と違って、splitとかreverseのような操作は出来ない

### 演算子
* 論理演算子
    * rubyで"偽"とは `nil`と`false`のこと
    * `&&` - 論理積, ``
    * `||` - 論理和,  ``
        * `nil || 50` = 50 (nilが偽, 50が真 なので) __trueやfalseを返す演算子ではない__

* 自己代入演算子
    * `&&=`
    * `||=` - __nilやfalseなら別途用意した値をセットする__ という用途に用いられる
        * `@a ||= generate_default_value` のような式を評価すると、変数 @a が真であるときには何もせず、@aが偽のときには generate_default_value メソッドを呼んでその戻り値でaを初期化

### クラス, オブジェクト


#### メソッド
* クラスメソッド(javaで言うstatic)は`def CLASSNAME.METHOD`形式で定義する
    * `def self.METHOD`とselfを使っても同意
        * __こちらの方が主流ぽい__ (クラス名変更時とかリファクタリングが少なく済む)

### モジュール
* __インスタンス化出来ないクラス__
    * ClassクラスはModuleクラスのサブクラス
* 2つの役割 = __mixin と 名前空間__

#### mixin
* __既存のクラスを拡張する時、拡張部分をモジュールとしてまとめておく という流儀がある__
* 2種類のメソッドを定義可能
    * モジュールメソッド - クラスで言うところのクラスメソッド(like static), `def self.method_name`で定義
    * (通常の)メソッド - __mixin用のメソッド__ `def method_name`で定義
* mixinしたメソッドを __インスタンスメソッドで__ 呼びたければ`include`
* mixinしたメソッドを __クラスメソッドで__ 呼びたければ`extend`

```ruby
# foo_module.rb
Module FooModule
  def self.module_method_name
  end

  def method_name
  end
end
```

```ruby
require "foo_module" # これ大事!!

class OtherClass
  include FooModule
end
OtherClass.new.method_name # これで呼べる

class OtherClass
  extend FooModule
end

OtherClass.method_name # これで呼べる
```

* __includeでもクラスメソッド形式で呼び出したい場合__ は、モジュールをネスト定義してその中にメソッドを定義する(よく見るのは`module ClassMethods`というモジュールを定義する形式)

```ruby
# your_module.rb
module YourModule
  def self.included(base)
    base.extend(ClassMethods)
  end
 
  module ClassMethods
    def hello
      puts 'Hello!'
    end
  end
 
  def bye
    puts 'Bye!'
  end
end
```
```ruby
require "your_module"

class YourClass
  include YourModule
end


y = YourClass.new
y.bye
YourClass.hello
```


#### 名前空間

#### 処理の取り込み mixin
[Rubyにおけるrequireとincludeとextend - yamazのRails日記 - Rubyist](http://rubyist.g.hatena.ne.jp/yamaz/20060722)

* `extend`
* `include`
* `autoload`


### ブロック, yield, Proc (は: P.138, パ: P.095)
* [[Ruby] ブロックとProcをちゃんと理解する - Qiita](http://qiita.com/kidachi_/items/15cfee9ec66804c3afd2)
    * それ単体では存在できず、メソッドの引数にしかなれない
        * 「do~endのカタマリ」がその辺に単体で転がっているのは見た事ないはず。
    * 引数として渡されたブロックは、yieldによって実行される
    * `yield`と書かず、ブロックであることを明示した書き方もある

    ```ruby
    # & はProcオブジェクトへの変換を表す
    def give_me_block(&block)
      # block.call でブロック内の処理を実行する
      block.call
    end

    # 上記を色々省略しまくってyieldだけで書いたメソッド
    def give_me_block_y
      # yield でブロック内の処理を実行する
      yield
    end

    # yieldは0個以上の式を取ることができ、それらの式の値をブロック呼び出し時にブロック引数として渡す
    # 呼び出し方
    #   foo_bar_baz do |item|
    #     p item
    #   end
    def foo_bar_baz
      yield "foo"
      yield "bar"
      yield "baz"
    end
    ```

    * __内部にyieldを持っている以上「ブロックを引数として受けとるメソッドである」ということを認識しないといけない__

#### Proc (は: P.140, パ: P.282)
* ブロックをオブジェクト化したものがProc
    * __ブロック引数は、仮引数の中で一番最後に定義されていなければならない__

* ブロックやProcを使えて何が嬉しいのか？
    * __メソッドを後々の利用時に柔軟に拡張出来る__
        * 「メソッドに対するメソッドのMix-in」
    * __クロージャとしての機能が得られる__

```ruby
# 5 いう数で好きなことをしてもらうメソッド
def magic_five_box(after_input, someProc)
  someProc.call(5, after_input)
end
```

### 例外処理begin...rescue...ensure
* `begin` = try
* `rescue` = catch
* `ensure` = finally
* `else` = 例外が一切発生しなかった場合に実行されるブロック


### lambda



### 引数（メソッド、ブロック）
* デフォルト引数 - `=`を使う. `def hoge(val = 'hoge')`
* 可変長引数 - `*`を使う. `def hoge(*params)`

#### キーワード引数
* [若手エンジニア／初心者のためのRuby 2.1入門（8）：Rubyの面白さを理解するためのメソッド、ブロック、Proc、lambda、クロージャの基本 (2/3) - ＠IT](http://www.atmarkit.co.jp/ait/articles/1409/29/news035_2.html)
* __キーワードに続けて:（コロン）を書き、その後にデフォルト値を続ける__

### %記法
[%記法](http://docs.ruby-lang.org/ja/2.2.0/doc/spec=2fliteral.html#percent)
    * `%w(foo bar baz)` - `['foo', 'bar', 'baz']` と同義

## ライブラリ
* pry
* itamae
* omniauth
* devise
* rspec-given
* rubocop - syntaxチェッカー

### Bundler
* [Gem - Bundler概要 - Qiita](http://qiita.com/hisonl/items/162f70e612e8e96dba50)

## ブクマ, ドキュメント


### ローカルでドキュメント閲覧
* `gem server`で`localhost:8808`にローカルドキュメントサーバが立つ
    * インストールしているgemに一覧が確認できる



## 疑問点

### xxx

## こんなツールが欲しい系

