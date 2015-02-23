## ブクマ
* [よく忘れる「rails generateコマンド」の使い方 - Rails Webook](http://ruby-rails.hatenadiary.com/entry/20140802/1406948695)
    * __scaffold は全部生成__

## railsについてよく聞く言葉・思想
* CoC = 「設定より規約」(Convention Over Configuration)
    * 「オレが5分でブログアプリ作ってやる」と宣言して`rails g scaffold`
* MVC
* TDD
* DRY（Don’t Repeat Yourself）
* [第1回 Railsを始めたきっかけ - ＠IT](http://www.atmarkit.co.jp/fcoding/rails/articles/passionate/01/passionate01b.html)


## とりあえずサクっと作りたい時にやること

### 事前準備
* rubyインストール
    * rbenvで
* bundlerインストール
* railsインストール
* ctags環境構築


### アプリを新規作成する
* `rails new APPNAME`
    * __`bundle install`だけ後でやりたければ`--skip-bundle`オプションを付ける__
        * gemのインストール先を例えば`vendor/bundle｀下にしたいときとか、こうするのが良さそう
        * __ctagsでのタグジャンプ使いたい＆ライブラリのソースにもジャンプしたいって時は、この手順にしてプロジェクトルートDIRで`ctags --langmap=Ruby:.rb -R .`する事になる（のかな？）
            * ってことはctags使いたければこの手順にすべきって事

#### ライブラリをrailsプロジェクト下にインストールする
[Gem - Bundler概要 - Qiita](http://qiita.com/hisonl/items/162f70e612e8e96dba50)

* `rails new`時に`--skip-bundle`した時(gemインストール先を変えたい時とかがそうていされる)の手順
* `bundle install --path vendor/bundle`
     * `--path`を付けなかった場合のデフォルト = rbenv使用時は`/Users/ユーザー名/.rbenv/versions/バージョン名/lib/ruby/gems/...`


### ルーティング仕様を決める
* __"何かを登録する" という機能の場合、登録用フォーム表示と登録処理は別にするのがパーフェクトrails流__
    * `/kinou/new` - 登録フォーム表示(GET)
    * `/kinou` - 登録(POST)
* 既存データ編集はPATCH/PUT, 削除はDELETE メソッドを受け付けるのがrails流っぽい

### コントローラ（とアクション）を作成する
* `bin/rails g controller CONTROLLERNAME ACTIONNAME` でコントローラ(とアクション)とビューを（とりあえず）生成する
* `config/route.rb`にルーティングを追加する

* 上記をまとめてやってくれる（ついでにモデルとマイグレーションの生成も）が __resource__ 、`bin/rails g resource ...`
* さらにさらにviewまで生成して、コマンド一発で簡易アプリが形になっちゃうのが __scaffold__ 、`bin/rails g scaffold ...`

### モデル、マイグレーションファイルを作成する
* `bin/rails g model MODELNAME COLUMN:TYPE...`
    * `db/migrate`ディレクトリにmigrationファイルが生成される
        * テーブル名は `MODELNAMEs` とモデル名に s を付けたもの
    * `app/model`下にモデルクラスが作成される
* migrationファイルに必要に応じてindexや制約(null制約とか)を追加する
* `bin/rake db:create` でDBを作成する
* `bin/rake db:migrate` - migaration適用
    * `bin/rake db:rollback` - migration戻し
        * __rollbackするときは、migrationファイルをmigrate実行時と同じ状態にしておかないとエラーになるので注意__
* もし既存テーブルの修正を行いたい場合は、`bin/rails g migration NAME`して修正用migrationを作成し、`bin/rake db:migrate`する
    * `up`メソッド - 適用前の状態を明示する
    * `down`メソッド - 適用後の状態を明示する
* `bin/rake db:migrate:status` <- migration適用/未適用の確認
* `sqlite3 db/development.sqlite3` <- DB確認
* `app/model/MODEL.rb`に必要に応じてバリデーションやビジネスロジックメソッド等を追加する

### ビューを作成する
* bootstrapとかで味気有るものにする
    * 公式サイトから落としてきて`vendor/assets/{stylesheets,javascripts}`配下へ展開
    * 展開したファイルを asset pipeline のリストに追加

### 動作確認
* `bin/rails s` <- 起動
* http://localhost:3000/APP へアクセス
* http://localhost:3000/rails/info/routes へアクセス <- ルーティング情報確認
    * http://localhost:3000/rails/info へアクセスしても同様( /routes へリダイレクトされる)
    * `bin/rake routes`コマンドでも確認可能


## 逆引き
デフォルト環境 = WEBrick(AP) + SQLite(DB)
__railsでデファクトなAPサーバーって何なのか調べる__

### 基本要素

#### Environments, Configurations
* `config/`下に配置
    * `application.rb`
    * `database.yml`
    * `secrets.yml`

* [設定ファイル(config) - Railsドキュメント](http://railsdoc.com/config)

#### Routing
* `config/route.rb`に定義

* [ルーティング(routes) - Railsドキュメント](http://railsdoc.com/routes)

##### REST

#### Controller
* `app/controllers/`下に配置
* コントローラは`ApplicationController`を継承する
    * `ApplicationController`は`ActionController::Base`を継承する
* `ApplicationController`には全アクション共通処理（ヘルパーとか）を書く

* [コントローラ(controller) - Railsドキュメント](http://railsdoc.com/controller)

#### ActiveRecord, Model
* railsのモデルとは？
    * nearly equal DB（のTBL/データ）の定義、所謂O/Rマッパー
    * __Modelクラスにはリレーション,バリデーション,スコープ,コールバック,enum,モデル独自の処理（メソッド）を定義する__
* `app/models/`下に配置
* モデルは`ActiveRecord::Base`を継承する
* __ActiveRecord はrails以外でも使える__

* [モデル(model) - Railsドキュメント](http://railsdoc.com/model), メソッド一覧もココ
    * CRUD関連
        * `new/build`
        * `save`
        * `create`
        * `find`
        * `find_by`
        * `where(COND, PARAMS)` - `COND`にはバインド変数`?`が使える
        * `count`
        * `order(SORT_COND)` - `SORT_COND`はSQLと同じように書ける。asc/descを省略したらasc
        * `update`
        * `destroy`
    * リレーション関連
        * `belongs_to`
        * `has_many`
        * `has_one`
    * 便利メソッド
        * `scope`
* [Rubyist Magazine - RubyOnRails を使ってみる 【第 3 回】 ActiveRecord](http://magazine.rubyist.net/?0006-RubyOnRails)

##### Migrations
* __データベースの構成を変える__
* `db/migrates/`下に配置
* マイグレーションは`ActiveRecord::Migration`を継承する

* [マイグレーション(migration) - Railsドキュメント](http://railsdoc.com/migration)

##### Associations
* __モデル間の関連を設定する__
    * `has_one`,`has_many`


##### Validations
* [検証(validation) - Railsドキュメント](http://railsdoc.com/validation)


#### View
* `app/views/`下に配置
* erb, haml, etc... でテンプレートを記述

* [ビュー(view) - Railsドキュメント](http://railsdoc.com/view)

#### Helper
* `app/helpers/`下に配置
* `module`として定義

#### Asset Pipeline
* `app/assets/`下に配置

+ [アセットパイプライン(Asset Pipeline) - Railsドキュメント](http://railsdoc.com/asset_pipeline)

#### RSpec


### 開発

#### OAuth認証
* Callback URLには`http://lvm.me:3000/auth/SERVICENAME/callback`といったURLを設定し、(画面遷移的には認証後ココに戻ってくるので)そこにcallback処理を実装する

#### セッション管理
* controllerの`session`オブジェクト
* `reset_session`メソッド

#### 色んなアクションに共通する事前/事後処理を定義したい
* アクションコールバック - `before_action`, `after_action`


#### viewの開発, asset pipeline



#### i18n, タイムゾーン関連



#### ユニットテストフレームワークを変える


#### ロギングのカスタマイズ
* `Rails.logger`
* `inspect`
* 出力先を変える
* 出力フォーマットを変える
* ログローテーション


#### gemを追加したい

#### vim
* [tpope/vim-rails](https://github.com/tpope/vim-rails)

#### （やりたい）
* `bin/rake routes`を __route.rbの変更を検知しつつ常時実行__ したい



### インフラ・ミドル

#### DBをMySQLに


##### 

#### WebサーバーをNginxに


#### SSLで動かす


#### Paas上で動かす

##### AWS


##### Heroku


## ライブラリ, 便利機能
[Railsなプロジェクトで利用している便利なGem一覧 - TIM Labs](http://labs.timedia.co.jp/2014/02/railsgem.html)

### ActiveRecord, Model

#### concern
* `ActiveSupport::Concern`
* __モジュール化促進、依存性の低減__ に便利
* 自作時は`module Foo extend ActiveSupport::Concern`というモジュールを定義
    * `included`ブロックを定義
    * `module ClassMethods`でクラスメソッド群を定義
* 自作モジュールを`Class Hoge include Foo`で取り込んで使う
* 利用例は [rails/configurable.rb at master · rails/rails](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/configurable.rb) が参考になる

#### active_hash
* [【Rails】静的データのモデル化にactive_hash gemが便利・・！ - kotatu.org](http://kotatu.org/blog/2014/10/09/active-hash-gem/)
    * 更新する必要のないデータを扱う時、DBテーブルを作ることなくyamlやJSON、RubyのハッシュをActiveRecordライクに扱える
    * 更にオブジェクトにhas_many等の関連付もできる
    * ActiveHashを使ってさっくりとモデル実装→必要になった時にActiveRecord化という開発ができて気持ちよさそう


## 疑問点

### ActiveRecord, Model
* [OK] __マスタデータを他TBLに保持するfooID, barID の2つが主キーになるような別TBL__ を設計する際、railsのモデル流なのは下記のどちら？__
    * fooID + barID がPKになるTBL
    * 別TBL独自のID がPKになるTBL __***こっちっぽい***__
        * __この場合削除は物理？論理？__
* [OK] `new`メソッドって何してる？
    * モデルオブジェクトを __生成__ するためのメソッド, DBに保存したければこの後に`save`を実行する
    * 生成・保存を一発でやるメソッドが `create`
* `has_many`のようなリレーション定義を行うクラスメソッドの引数（モデル名）、何でUpper caseじゃなくても良いの？(例えば`:books`みたいな)
* Relationクラスでは "SQLは即座に実行されず、組み立てが完了した時点で実行される" とあるが、例えば`where`メソッドの後にさらに付加条件があるか・もう条件が無いか、はコードをコンパイル？した時点にならないと分からないはずだが、どういう仕組なんだろう？
    * クエリの構築はメソッドチェーンで行うのがActiveRecord
    * `to_a`メソッドを明示的に呼ぶとその時点でSQLが発行される, __このとき返ってくる型はActiveRecord::Relationではない(モデルの配列で返ってくる)__
* 機能追加時のデプロイタイミングで、新しいモデルの追加作成が必要な場合、どういうデプロイ運用を行っているのか？
    * デプロイ -> 本番鯖に入ってmigrationの各コマンドを叩く とかじゃないよなぁまさか...
* [OK] `craete`失敗時(validation違反とかで)のログ、何で失敗したかわかりづらくね？(consoleから実行した時の事)
    * `errors`というオブジェクトにエラー情報が保存される
        * `full_messages`メソッドでメッセージだけを確認可能
        * が、これはActiveRecordオブジェクト経由での失敗時。クラスメソッドを呼んで失敗した場合は何処にも保存されない
* `rails g model`はどういう時に行うもの？
    * 逆にどういう時なら`rails g scaffold`でも良いのか？
* `rake db:create`を行うケースと行わないケースの違いは？
    * `rake db:migrate`しか行わないケース有るよね
* [OK] てか何か scaffold が固まって動かなくなり始めた
    * [AWS(EC2)+Rails+MySQLでrails generate が動かない？ - To create, To entertain](http://tkoyama1988.hatenablog.com/entry/2014/06/08/192614)
    * [【Ruby On Rails】　「rails generate」が動かない | approad](http://app.road.jp.net/?p=1679)
    * `spring`が悪さしてたっぽい（ __TODO springって何ぞやから詳しく調べる__ )
        * `bin/spring stop` -> `bin/spring` したらOK
* `bin/rake db:create` でDBを作成する時に何か`db/development.sqlite3 already exists`ってエラーが出た
    * __このrakeタスクで何が実行されてるのか詳しく調べる__
* [OK][パー本 p.204] commentは`allow_blank: true`じゃなくて`null: true`じゃダメなの？
    * NULLを許可するか？と、空文字を許容するか？、を何かしらの理由で分けて考えてる？
        * 空文字じゃなくてNULLにすると、viewに`<NULL>`みたいに表示されちゃうとか？
        * まあ __MySQLだとNULLと空文字は扱い違う__ し、`NOT NULL DEFAULT ''`とかやるし妥当か
* 「ユーザーがあるイベントに参加済だったら、参加するボタンを非表示に」って事をやりたい場合、参加済かどうかを判定する処理を何処に・どう書くのがrails流なのかがよく分からない
* [パー本 p.208] ステータスコードを返した後の処理を何でjsで書くのか？（そもそもこのjs作成をサボると、送信ボタン押下してもモーダルが閉じなかったり挙動が変になるが）
* [Ruby - てめえらのRailsはオブジェクト指向じゃねえ！まずはCallbackクラス、Validatorクラスを活用しろ！ - Qiita](http://qiita.com/joker1007/items/2a03500017766bdb0234) を読んで思ったけど、結局どういう設計がベターなわけ？(ベスト ではない。そんなものはプロジェクトによってマチマチだから)
    * "__一つのクラスとして独立してテストコードを書きたいかどうか、というのも私の中では指標の一つ__"
    * "Concern便利だけど、何でもかんでもConcern使えばいいってわけでもない"
    * "Validatorクラスは通常一度しかインスタンス化されず、そのオブジェクトをずっと使い回すことになる"

### routeとController
* [パー本 p.69] `rails g model``rails g controller`した後に「ルーティング情報を削除してから定義」みたいな事書いてるけど、削除対象になるような記述が最初から無かった
* [OK][パー本 p.179] `uninitialized constant SessionController`ってエラーで動かない
    * route.rbの`get '/auth/:provider/callback' => 'sessions#create'`, このコントローラ名を`sessions`ではなく`session`にtypoしてただけだった
* `routes.rb`の`resouces`って具体的に何？他とどう違うの？
* [OK][パー本 p.187] `current_user.created_events.build`の下りがいまいち理解できてない。`created_events`はUserモデルに has_many関連 として定義してるんだけど、`build`って何処でどういう処理として定義されてるメソッドなの？とか
    * これだ！ [build - リファレンス - Railsドキュメント](http://railsdoc.com/references/build), モデルのメソッド。"モデルオブジェクトを生成する, saveメソッドなどを使って保存する"
* [パー本 p.199] updateメソッド、`params[:id]`と`event_params`(実体はメソッドで`:event`ってパラメータを参照してる)の2つのパラメータを使ってるけど、これ複数パラメータってどうやって受け付けてるんだろ？
* [OK][パー本 p.204] create_tickets.rbの生成のされ方が本と異なる(`add_foreign_key`が定義されてる)
    * [rails/4_2_release_notes.md at master · rails/rails](https://github.com/rails/rails/blob/master/guides/source/4_2_release_notes.md) の __Foreign Key Support__ に書いてあった
* [パー本 p.204] リスト6.38、何でadd_indexを2つ書いてるの？（カラムの並びに意味がある？)


### view
* [パー本 p.180] erbで使ってる`logout_path`って他の何処にも定義されていない変数(?)なんだけど、どうやって認識してるの？
    * `route.rb`の`Prefix`名 + `_path` って形式？（に見える）
    * "The Rails 4 Way" の p.49(2.4.1 Creating a Named Route) も合わせて読みたい
* [パー本 p.188] `<%= msg %>`の部分でvimが"possibly useless use of a variable in void context"ってエラー吐くの何で？
    * __エラー無視しても一応ちゃんと動いてる__
* view(erb, html)のvimでの開発効率もうちょっとどうにかしたい
* [パー本 p.201] `f.submit`で生成されるformタグのaction URLって勝手に良しなに動かしたいアクションを選んでくれてるの？？
* 結局erbがベターなの？
* vimでerb書く時HTMLの編集が面倒くさいのどうにかしたい。自動閉じタグとか諸々欲しい


### 漏洩や他社閲覧させたくない情報の管理について
* `config/secrets.yml`に書け、ってことだけど、ココに書けば絶対大丈夫なの？（環境変数使うとかも考えてたんだけど）
    * パーフェクトrails p.173 に書いてた
* `config/secrets.yml`で使える書き方・記号の意味って？

### config/initializersについて
* そもそもこれは何？
* 書き方は？

### omniauth
* パーフェクトrailsp.176 の例で動かすとエラー(401 Authorization Required)になる

### 設計(特にfat {model,controller}防止)
* プロジェクト肥大化時とかにデフォルトのディレクトリ構成からいくつかディレクトリを追加して(用途別にクラスを切り分けるとかして)見やすい構成にしたい場合、その追加してディレクトリのソースをrailsに読み込んでもらう為には何をすればよいか？
    * concers 使おうず、って話。（concersに頼りすぎるのは危険 派もいる）
    * ネームスペース切って整理する話(カラム数の多いユーザー属性 とか) : [RailsでModelをDBから分割・整理する - リア充爆発日記](http://d.hatena.ne.jp/ria10/20150101/1420123705)
        * [プライマリキーを使った1:1関連でカラム数の多いテーブルを分割する - Hidden in Plain Sight](http://kenn.hatenablog.com/entry/2014/03/05/081525)

### 運用
* 大規模なサービスでMySQL使ってたりして、ALTER TABLEの負荷やコストが馬鹿にならないにも関わらず`db:migrate`でデプロイ時にALTERするのって、やばい時あるんじゃね？
    * クックパッドの事例, `db:migrate`は使ってない - [クックパッドにおける最近のActiveRecord運用事情 - クックパッド開発者ブログ](http://techlife.cookpad.com/entry/2014/08/28/194147)

### railsのソース
* 例えばactivesupportの下の複数のソースファイルに __Moduleによる名前空間分割無しで__ `Numeric`というクラスが複数定義されているんだけど、これはrails実行時に内部でどういう動きをしているんだろう？(読み込まれ方とか)
    * rubyのNumericクラスを拡張している
        * __rubyはオープンクラスなので直接クラスを拡張しても構わない__

### その他
* デフォルトの`.gitignore`、何でlogディレクトリをignoreしてないの？
    * `/log/*`になってるからっぽい。先頭の `/`が余計
    * railsのプロジェクトをgit commitするとき、rails newでデフォルト生成される`.gitignore`だけiginoreすりゃOK？
* railsのエラー画面の下部のコンソールっぽいやつ何？何に使う？
* `rails g HOGEHOGE` HOGEHOGEに何を使うかの選択基準って結局何なの？
* [OK] Gemfileで定義してインストールするライブラリとそのソースってどこに格納されるの？
    *  [Gem - Bundler概要 - Qiita](http://qiita.com/hisonl/items/162f70e612e8e96dba50) を見るべし


## こんなツールが欲しい系
* プロジェクト成果物のメトリクスを取るツール。モデル数とかビュー数とかetc

## ソースコード リーディング
[rails/rails](https://github.com/rails/rails)


* ディレクトリ構成
    * 機能別ディレクトリがある
        * `activemodel`
        * `activerecord`
        * `activesupport`
        * `activejob`
        * `actionpack` - controllerはココ
        * `actionview`
        * `actionmailer`
    * `機能名/lib/機能名.rb`
    * `機能名/lib/機能名/HOGE.rb`

### ActiveRecord

### Controller
* `respond_to`メソッド - [rails/mime_responds.rb at master · rails/rails](https://github.com/rails/rails/blob/master/actionpack/lib/action_controller/metal/mime_responds.rb#L186)

## 日々

### 2015-02-10
* `rails generate`
    * `scaffold TBLNAME COLUMN:TYPE...`
        * データの更新や追加などの __テンプレートをひと通り生成する__
            * __scaffold すれば簡単んアプリはすぐ出来る__
    * `model MODELNAME COLUMN:TYPE...`
        * __機能追加時はmodelをgenerate__
        * 多対多表現用の中間モデルを作りたい場合はmigrationではなくこちらのmodelを使う
    * `migration AddXToY REFNAME:references`
        * YモデルにXモデルへの参照を追加
        * `AddXToY`というマイグレーションクラスが作成される
        * とにかく __DBへの一連の変更操作を行う場合は、まずmigrateを行う__ というのがrailsの決まり事。リレーションの定義なり・カラムの追加なり・属性変更なり
            * そして __これらは実行(`up`)も出来るし戻し(`down`)も出来る__
            * 何故なら `db/migrate`下に適用したmigrationがrubyファイルで保存されてそこで管理されているから
    * `controller NAME ACTIONS...` - 後述
* `rails destroy` - migrationの削除。例えばgenerateで名前を間違えた、とかいう場合に一旦削除したい場合とか
* `rake`
    * `db:create`
    * `db:migrate`
        * __`rails g migration`して`rake db:migrate` これが定型的なオペレーション__
        * 何かおかしいなと思ったら`db:migrate:status`して`down`のままになってるmigrationが無いか確認すると良い
    * `db:migrate:status`
    * `routes`
* ActiveRecordの使い方
    * CRUD
        * C - `create`
            * `new` -> `save`
        * U - `find` -> `(値変更)` -> `save`
        * D - `destroy`
        * 更新系メソッドには末尾に`!`が付く版が用意されている。こちらはバリデーション失敗時に例外を投げる
    * 条件指定クエリ（を発行するためのメソッド）
        * `where`でlike文を使う場合、部分一致記号`%`はパラメータ指定する文字列に含める必要あり
    * よく使う条件に名前を付けて定義
        * `scope`メソッド
            * __scopeメソッドの第2引数は関数（メソッド）__
                * この引数は動的なパラメータ
                * この書き方どっかで見たこと有るなと思ったらcoffeeだった
        * `default_scope`メソッド
    * リレーションの定義
        * `has_many`メソッド
        * `belongs_to`メソッド
    * バリデーション
        * `validates`メソッド
            * `presence` - 必須項目
            * `length` - 長さ
                * `maximum` - 最大長
            * `numericality` - 数値
        * `errors`オブジェクト - save失敗時のエラー情報
        * `valid`メソッド - バリデーションだけを行う, 同じく`errors`にエラー情報が設定される
        * `validate`ブロック - 複雑なバリデーションはココに書く
    * コールバック
        * `before_validation` - save前に行う前処理。チェックとか変換とか
        * `after_destroy` - destroy後に行う後処理。ログ保存とか
        * `if``unless`で、コールバックの起動条件を指定可能
* `rails c`で作成したクラスの動作確認する際、クラスの変更を反映させるには一旦consoleを抜けて入り直す必要あり

### 2015-02-12

#### ActiveRecord
* enum
    * ステータス系やコード系の項目で使う
    * 0ベースインデックス, これを変えたければ`NAME:VALUE`形式でハッシュで明示する
    * __コード上では文字列で指定しつつ、ActiveRecord側では数値に変換して保存することが出来る__
        * 文字列じゃなくてシンボルでもいい
    * `MODEL_OBJ.ENUM_NAME?`メソッドで、そのモデルオブジェクトがENUM_NAMEという属性かどうかをtrue/falseで取得可能
    * `MODEL_OBJ.ENUM_COLUMN`メソッドで、そのモデルオブジェクトのenum値を __文字列で__ 取得可能

#### Controller
* ルーティング＋アクションの定義（追加）時の流れ
    * `config/route.rb` にルーティングを定義
    * `rails generate controller NAME`してcontrollerを（無ければ）追加
    * controllerにアクションメソッドを定義
* `ApplicationContoroller`クラス
    * `respond_to`メソッド
    * アクションにコールバックを定義可能
        * `before_action`メソッド etc
            * `only``except`オプション - 特定のアクションのみ/を除外 する設定
        * `skip_before_action`メソッドetc - `skipXXX`でコールバックの実行をスキップする
    * Mass Assignment と StrongParameters
* `route.rb`
    * `resources``resource`オプション - GET(show),POST(create),PUT(update),DELETE(destroy)という各アクションを1行で定義出来る
        * __RESTfulなアプリを作ることを意図している__
        * resourcesをブロック化して入れ子にすることも可能
            * `member`ブロック - 個別のリソースに対するアクションを設定(detail みたいな)
            * `collection`ブロック - 全体のリソースに対するアクションを設定(search みたいな)
    * 本や著者のようにアプリ上に複数存在するリソースを扱う場合は`resources`で、ログインユーザーのプロフィール等1つしか存在しないものは単数形の`resource`を使う
* 例外処理
    * __特定の例外を投げるとそれに対応したステータスコードを返す__ のがrails
    * `rescue_from`メソッド - 特定の例外を特定の処理に紐付ける
* view
    * `render`メソッド
    * erbファイル
    * `redirect_to`メソッド
    * partialテンプレート - 共通で使い回したい部分テンプレートのこと
        * `_hoge.html.erb`というようにファイル名を`_`で始める
        * 使いたい側(のテンプレート)では`<%= render partial: 'hoge' %>`と呼び出す
    * layout
        * `app/views/layouts`下に配置する、レイアウトで使用するテンプレート
            * よくある例だと共通ヘッダ・フッタとか
            * デフォルトでは`application.html.erb`というファイルが用いられる
        * layoutファイル内で`<%= yield %>`すると、そこに呼び出し元のテンプレートが埋め込まれる
            * ヘッダ・フッタの例なら、呼び出し元はbody部しか書かない
    * variants - テンプレートの切り替え
        * PC,SP,FPでテンプレを分ける場合 とか
    * ヘルパー - フォーマット変換やタグの構築を行うヘルパーメソッド群
        * erbではメソッド名をそのまま記述して呼び出せる
        * irb(controllerとかmodelとかでも？)では`helper.METHOD`で呼び出す
        * 独自ヘルパーは`app/helpers`に定義する
            * `application_helper.rb` - アプリ全体で使うヘルパー
            * `CONTROLLER_helper.rb` - 各コントローラ毎に使うヘルパー
     * XSS対策
        * railsのビュー処理は自動で文字列のXSSエスケープを行う
        * エスケープを回避したければ`raw`ヘルパーメソッドを使う
     * jbuilder (JSON描出用プラグイン)
        * `app/views/CONTROLLER/ACTION.json.jbuilder`ファイルに定義する
        * `json.extract! MODEL, COLUMNS...`メソッド - 普通のjson描画ならコレ
            * `!`で終わるメソッドは特殊な意味を持つ
            * __逆に`!`で終わらないメソッドは、jsonのキーとして扱われる
        * rubyのコードがそのまま書ける

### 2015-02-15
* `rails new -T`オプション - `Test::Unit`ファイル群の生成をスキップする
    * RSpec使いたいときはコレ

#### OAuth(twitter)認証を導入する
* __`lvm.me` 127.0.0.1を指すドメイン__
* OmniAuth というgemを使う, 認証フレームワーク のようなgem
    * どのサービスを使った認証を利用するか？は別gemになってる(omniauth-twitter)
    * __`Gemfile`にgemを追記して`bin/bundle`する__
    * `config/secrets.yml`にtoken等を追記
    * `config/initializers/omniauth.rb`を作成
* twitterログイン ボタンを画面に追加
* 一旦rackを終了して再起動して上記ボタンへアクセス（`/auth/twitter`）
    * この時点ではcallbackが無いのでエラー
* ログイン兼ユーザ登録のコントローラを作成
    * `request.env['omniauth.auth'])`という変数にユーザ情報やアクセストークンが格納されている

#### ActiveRecord
* `find_or_create_by`メソッド

#### Controller
* `helper_method METHODNAME` でControllerのメソッドをヘルパーとして使う
    * => view内でもこのメソッドが呼べる

#### View
* トップページを独自画面にカスタマイズする
    * `rails g controller welcome index`
    * `config/route.rb`で`root`メソッドで定義する `root to: 'welcome#index'`
* bootstrapを導入する
    * 落としてきて`vendor/assets`配下へ
    * asset pipiline追加
        * `app/assets/{javascripts,stylesheets}/application.{js,css}`
    * layout(例えばヘッダ)に適用してみる
        * `app/views/layouts/application.html.erb`
* `flash[:notice]``<% if flash[:notice]` 文 - flashメッセージを表示する


### 2015-02-16

#### その他
* タイムゾーン設定, `config/application.rb` - `config.time_zone = 'Tokyo'` で日本時間
* フォームのラベルやエラーメッセージのi18n対応
    * `config/application.rb` - `config.i18n.default_locale = :ja` で日本語化
    * `config/locales/ja.yml'に辞書データのyamlを配置`
        * 辞書データは rails-i18n のリポジトリにまとめられている
        * `curl -o config/locales/ja.yml -L https://raw.github.com/svenfuchs/rails-i18n/master/rails/locale/ja.yml`
        * これとは別に独自辞書データも追加したい場合は、例えば`ar_ja.yml`のような別ファイルに定義すると管理しやすい
    * 反映時は`rails s`再起動が必要っぽい
    * (パラメータ内容とかで動的にlocaleを変更したい場合の例は "The Rails 4 Way"のp.378参照)
* 現在日次の取得 = `Time.zone.now`
* `rails g resource`コマンド

* [Rails Hub情報局: Rails4に間に合うか、REPL付きエラー画面「Better Errors」がイイ感じ](http://el.jibun.atmarkit.co.jp/rails/2012/12/rails.html)
    * Rackで動くアプリのエラー画面下部にREPLが表示される
    * [Railsのweb-consoleについて | 日々雑記](http://y-yagi.tumblr.com/post/96428108880/rails-web-console)

#### ActiveRecord
* `A氏が作ったB案`みたいに、2TBL間のリレーションが存在するデータを取得したい場合、`user.RELATION.find(params[:id])`のように、一度`has_many`等で設定したリレーションを挟んでCRUDメソッドを呼ぶのが流儀

#### Model
* `validates COLUMN, VALIDATEs...` - バリデーション
    * validate METHODNAME` - 複雑なバリデーションはメソッド化(2項目間の関連チェックとか)`
        * 最後のsが無いの、紛らわしいので注意
* `errors.add(COLUMN, MSG)` - バリデーションNG時等に対象項目にエラー（とメッセージ）を追加する
* `has_many`


####Controller
* `rails g resource` - `routes.rb`に`resources RESOURCENAME`形式でリソースが追加される
* `params`変数 - リクエストパラメータ関連の処理を行うオブジェクト
    * `params.require(NAME)` - 必須パラメータ指定
        * `params.require(NAME).permit(COLUMNS..)` - NAMEパラメータ内のkey-valueのうち許可するkeyパラメータを指定
* `session` - セッション情報
    * ログイン状態を管理する処理とかで使う

#### View
* `form_for(MODEL, class: CLASS, role: ROLE) do |f| ...` - 指定したモデルの情報を元にformを作るためのヘルパー
    * __viewで使用する際は`<%= form_for ... %>`と`<%= %>`構文で呼び出す__
    * `f.label`
    * `f.text_field`
    * `f.datetime_select`
    * `f.submit`
        * `disable_with`オプション - 多重送信防止


### 2015-02-18

#### その他

#### ActiveRecord
* migrationの適用・戻し・修正・確認のやり方
* `belongs_to`(多側)と`has_many`(一側)はワンセットで定義
    * `has_many`で指定する名前は複数形にするのが流儀

#### Model
* `include`メソッド
* N+1 問題


#### Controller

#### View
* モーダル表示 with bootstrap
    * モーダルでの処理後、__javascriptで__ レスポンス判定と画面描画を行う

### 2015-02-23

#### その他
* `rake stats`でプロジェクトの統計情報が見れる(controller数とか)

#### ActiveRecord
* [開発現場でちゃんと使えるRails 4入門（5）：ActiveRecordの基本機能とマイグレーション、バリデーション (2/3) - ＠IT](http://www.atmarkit.co.jp/ait/articles/1405/30/news036_2.html)
    * `where(...).order(...)`といった具合にメソッドチェインで呼べるのは便利
* `has_many ..., dependent: :destroy` - 親が削除されたら子も削除
* `dependent: :nullify`
* `before_destroy`コールバック

#### Model

#### Controller
* route.rbでの単数形`resource`・複数形`resources`の使い分け
* route.rbでの __`resource`に入れ子になった`get`__
    * `resource`(`resources`)で自動生成されるやつ以外のアクションを追加する時に使う
* `reset_session`セッション情報の初期化(ログアウト時とか)
* `rescue_from`によるコントローラ内での例外捕捉
* routes.rbので「未定義URL全てをキャッチするルーティング」を定義してエラー捕捉する `match '*path' => NAME, via: :all`

#### View


### 2015-02-xx

#### その他

#### ActiveRecord

#### Model

#### Controller

#### View

