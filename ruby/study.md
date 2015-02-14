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