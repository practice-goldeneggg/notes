## とりあえずサクっと作りたい時にやること

* `rails new APPNAME`
* `cd APPNAME`
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

### 開発

#### 色んなアクションに共通する事前/事後処理を定義したい
* アクションコールバック - `before_action`, `after_action`

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

### その他
* デフォルトの`.gitignore`、何でlogディレクトリをignoreしてないの？
    * `/log/*`になってるからっぽい。先頭の `/`が余計

## こんなツールが欲しい系
* プロジェクト成果物のメトリクスを取るツール。モデル数とかビュー数とかetc
