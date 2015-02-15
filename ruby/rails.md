## ブクマ
* [よく忘れる「rails generateコマンド」の使い方 - Rails Webook](http://ruby-rails.hatenadiary.com/entry/20140802/1406948695)
    * __scaffold は全部生成__


## とりあえずサクっと作りたい時にやること

* `rails new APPNAME`
* `cd APPNAME`
* 機能概要を決める
    * ルーティング仕様を決める
        * __"何かを登録する" という機能の場合、登録用フォーム表示と登録処理は別にするのがパーフェクトrails流__
            * `/kinou/new` - 登録フォーム表示(GET)
            * `/kinou` - 登録(POST)
        * 既存データ編集はPATCH/PUT, 削除はDELETE メソッドを受け付けるのがrails流っぽい
    * コントローラ（とアクション）
        * `bin/rails g controller CONTROLLERNAME ACTIONNAME` でコントローラ(とアクション)とビューを（とりあえず）生成する
        * `config/route.rb`にルーティングを追加する
* モデルを決める・設計する
    * __マスタデータを他TBLに保持するfooID, barID の2つが主キーになるような別TBL__ を設計する際、railsのモデル流なのは下記のどちら？__
        * fooID + barID がPKになるTBL
        * 別TBL独自のID がPKになるTBL __***こっちっぽい***__
            * __この場合削除は物理？論理？__
    * `bin/rails g scaffold MODELNAME COLUMN:TYPE...`
        * `db/migrate`ディレクトリにmigrationファイルが生成される
        * テーブル名は `MODELNAMEs` とモデル名に s を付けたもの
        * てか何か scaffold が固まって動かなくなり始めた
            * [AWS(EC2)+Rails+MySQLでrails generate が動かない？ - To create, To entertain](http://tkoyama1988.hatenablog.com/entry/2014/06/08/192614)
            * [【Ruby On Rails】　「rails generate」が動かない | approad](http://app.road.jp.net/?p=1679)
            * `spring`が悪さしてたっぽい（ __TODO springって何ぞやから詳しく調べる__ )
                * `bin/spring stop` -> `bin/spring` したらOK
        * モデルを作る - `rails g model MODELNAME COLUMN:TYPE...`
            * `app/model`下にモデルクラスが作成される
    * `bin/rake db:create`
        * DBを作成するんだと思うんだけど、何か`db/development.sqlite3 already exists`って出た
            * __このrakeタスクで何が実行されてるのか詳しく調べる__
    * `bin/rake db:migrate`
    * `sqlite3 db/development.sqlite3` <- DB確認
    * `bin/rake db:migrate:status` <- migration適用/未適用の確認
* ビューをbootstrapとかで味気有るものにする
    * 公式サイトから落としてきて`vendor/assets/{stylesheets,javascripts}`配下へ展開
    * 展開したファイルを asset pipeline のリストに追加
* `bin/rails s` <- 起動
* http://localhost:3000/tasks へアクセス
* http://localhost:3000/rails/info/routes へアクセス <- ルーティング情報確認
    * http://localhost:3000/rails/info へアクセスしても同様( /routes へリダイレクトされる)
    * `bin/rake routes`コマンドでも確認可能


## 逆引き
デフォルト環境 = WEBrick(AP) + SQLite(DB)
___railsでデファクトなAPサーバーって何なのか調べる___

### インフラ・ミドル

#### DBをMySQLに


#### WebサーバーをNginxに


#### SSLで動かす


#### Paas上で動かす

##### AWS


##### Heroku


### 開発

#### OAuth認証
* Callback URLには`http://lvm.me:3000/auth/SERVICENAME/callback`といったURLを設定し、(画面遷移的には認証後ココに戻ってくるので)そこにcallback処理を実装する

#### セッション管理
* `session`オブジェクト
* `reset_session`メソッド

#### 色んなアクションに共通する事前/事後処理を定義したい
* アクションコールバック - `before_action`, `after_action`

#### viewの開発, asset pipeline



#### ユニットテストフレームワークを変える


#### ロギングのカスタマイズ
* `Rails.logger`
* `inspect`
* 出力先を変える
* 出力フォーマットを変える
* ログローテーション


#### gemを追加したい


## 疑問点

### ActiveRecord, モデル
[モデル(model) - Railsドキュメント](http://railsdoc.com/model)

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

### routeとController
* [パー本 p.69] `rails g model``rails g controller`した後に「ルーティング情報を削除してから定義」みたいな事書いてるけど、削除対象になるような記述が最初から無かった
* [OK][パー本 p.179] `uninitialized constant SessionController`ってエラーで動かない
    * route.rbの`get '/auth/:provider/callback' => 'sessions#create'`, このコントローラ名を`sessions`ではなく`session`にtypoしてただけだった


### view
* [パー本 p.180] erbで使ってる`logout_path`って他の何処にも定義されていない変数(?)なんだけど、どうやって認識してるの？
    * `route.rb`の`as`属性で定義した名前 + `_path` って形式？



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

## こんなツールが欲しい系
* プロジェクト成果物のメトリクスを取るツール。モデル数とかビュー数とかetc

## ソースコード リーディング

### ActiveRecord

### Controller
* `respond_to`メソッド - [rails/mime_responds.rb at master · rails/rails](https://github.com/rails/rails/blob/master/actionpack/lib/action_controller/metal/mime_responds.rb#L186)
