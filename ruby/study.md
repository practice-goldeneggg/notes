## study

### rails

#### 2015-02-10
* railsのモデルとは？
* `rails generate`
    * `scaffold TBLNAME COLUMN:TYPE...`
    * `model MODELNAME COLUMN:TYPE...`
        * 多対多表現用の中間モデルを作りたい場合はmigrationではなくこちらのmodelを使う
    * `migration AddXToY REFNAME:references`
        * YモデルにXモデルへの参照を追加
        * `AddXToY`というマイグレーションクラスが作成される
        * とにかく __DBへの一連の変更操作を行う場合は、まずmigrateを行う__ というのがrailsの決まり事。リレーションの定義なり・カラムの追加なり・属性変更なり
            * そして __これらは実行(`up`)も出来るし戻し(`down`)も出来る__
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
