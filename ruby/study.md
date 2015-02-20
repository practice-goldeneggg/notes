## study

### ruby

#### 2015-02-12
* [%記法](http://docs.ruby-lang.org/ja/2.2.0/doc/spec=2fliteral.html#percent)
    * `%w(foo bar baz)` - `['foo', 'bar', 'baz']` と同義


### rails

#### 2015-02-10
* railsのモデルとは？
    * __nearly equal DB（のTBL/データ）__
    * Modelクラスにはリレーション,バリデーション,スコープ,コールバック,enum,モデル独自の処理（メソッド）を定義する
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

#### 2015-02-12

##### ActiveRecord
* enum
    * ステータス系やコード系の項目で使う
    * 0ベースインデックス, これを変えたければ`NAME:VALUE`形式でハッシュで明示する
    * __コード上では文字列で指定しつつ、ActiveRecord側では数値に変換して保存することが出来る__
        * 文字列じゃなくてシンボルでもいい
    * `MODEL_OBJ.ENUM_NAME?`メソッドで、そのモデルオブジェクトがENUM_NAMEという属性かどうかをtrue/falseで取得可能
    * `MODEL_OBJ.ENUM_COLUMN`メソッドで、そのモデルオブジェクトのenum値を __文字列で__ 取得可能

##### Controller
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

#### 2015-02-15
* `rails new -T`オプション - `Test::Unit`ファイル群の生成をスキップする
    * RSpec使いたいときはコレ

##### OAuth(twitter)認証を導入する
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

##### ActiveRecord
* `find_or_create_by`メソッド

##### Controller
* `helper_method METHODNAME` でControllerのメソッドをヘルパーとして使う
    * => view内でもこのメソッドが呼べる

##### View
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


#### 2015-02-16

##### その他
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

##### ActiveRecord
* `A氏が作ったB案`みたいに、2TBL間のリレーションが存在するデータを取得したい場合、`user.RELATION.find(params[:id])`のように、一度`has_many`等で設定したリレーションを挟んでCRUDメソッドを呼ぶのが流儀

##### Model
* `validates COLUMN, VALIDATEs...` - バリデーション
    * validate METHODNAME` - 複雑なバリデーションはメソッド化(2項目間の関連チェックとか)`
        * 最後のsが無いの、紛らわしいので注意
* `errors.add(COLUMN, MSG)` - バリデーションNG時等に対象項目にエラー（とメッセージ）を追加する
* `has_many`


##### Controller
* `rails g resource` - `routes.rb`に`resources RESOURCENAME`形式でリソースが追加される
* `params`変数 - リクエストパラメータ関連の処理を行うオブジェクト
    * `params.require(NAME)` - 必須パラメータ指定
        * `params.require(NAME).permit(COLUMNS..)` - NAMEパラメータ内のkey-valueのうち許可するkeyパラメータを指定
* `session` - セッション情報
    * ログイン状態を管理する処理とかで使う

##### View
* `form_for(MODEL, class: CLASS, role: ROLE) do |f| ...` - 指定したモデルの情報を元にformを作るためのヘルパー
    * __viewで使用する際は`<%= form_for ... %>`と`<%= %>`構文で呼び出す__
    * `f.label`
    * `f.text_field`
    * `f.datetime_select`
    * `f.submit`
        * `disable_with`オプション - 多重送信防止


#### 2015-02-18

##### その他

##### ActiveRecord
* migrationの適用・戻し・修正・確認のやり方
* `belongs_to`(多側)と`has_many`(一側)はワンセットで定義
    * `has_many`で指定する名前は複数形にするのが流儀

##### Model
* `include`メソッド
* N+1 問題


##### Controller

##### View
* モーダル表示 with bootstrap
    * モーダルでの処理後、__javascriptで__ レスポンス判定と画面描画を行う


#### 2015-02-xx

##### その他

##### ActiveRecord

##### Model

##### Controller

##### View

