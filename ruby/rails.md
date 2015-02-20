## ブクマ
* [よく忘れる「rails generateコマンド」の使い方 - Rails Webook](http://ruby-rails.hatenadiary.com/entry/20140802/1406948695)
    * __scaffold は全部生成__


## とりあえずサクっと作りたい時にやること

### アプリを新規作成する
* `rails new APPNAME`

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
* `app/models/`下に配置
* モデルは`ActiveRecord::Base`を継承する

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

##### Validations
* [検証(validation) - Railsドキュメント](http://railsdoc.com/validation)

##### Migrations
* `db/migrates/`下に配置
* マイグレーションは`ActiveRecord::Migration`を継承する

* [マイグレーション(migration) - Railsドキュメント](http://railsdoc.com/migration)

##### Associations

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


#### WebサーバーをNginxに


#### SSLで動かす


#### Paas上で動かす

##### AWS


##### Heroku



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


### 漏洩や他社閲覧させたくない情報の管理について
* `config/secrets.yml`に書け、ってことだけど、ココに書けば絶対大丈夫なの？（環境変数使うとかも考えてたんだけど）
    * パーフェクトrails p.173 に書いてた
* `config/secrets.yml`で使える書き方・記号の意味って？

### config/initializersについて
* そもそもこれは何？
* 書き方は？

### omniauth
* パーフェクトrailsp.176 の例で動かすとエラー(401 Authorization Required)になる


### その他
* デフォルトの`.gitignore`、何でlogディレクトリをignoreしてないの？
    * `/log/*`になってるからっぽい。先頭の `/`が余計
    * railsのプロジェクトをgit commitするとき、rails newでデフォルト生成される`.gitignore`だけiginoreすりゃOK？
* railsのエラー画面の下部のコンソールっぽいやつ何？何に使う？
* `rails g HOGEHOGE` HOGEHOGEに何を使うかの選択基準って結局何なの？


## こんなツールが欲しい系
* プロジェクト成果物のメトリクスを取るツール。モデル数とかビュー数とかetc

## ソースコード リーディング

### ActiveRecord

### Controller
* `respond_to`メソッド - [rails/mime_responds.rb at master · rails/rails](https://github.com/rails/rails/blob/master/actionpack/lib/action_controller/metal/mime_responds.rb#L186)
