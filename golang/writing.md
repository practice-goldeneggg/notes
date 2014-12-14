## goコードの書き方
とりあえず下記を読んどくべし

* [How to Write Go Code - The Go Programming Language](https://golang.org/doc/code.html)
* [Goコードの書き方 - golang.jp](http://golang.jp/code)

### ワークスペースの構成
* [golang - 【翻訳】プロダクション環境でのベストプラクティス - Qiita](http://qiita.com/umisama/items/c2a8db6c23db18dd5437) の開発環境についての節を参照
* __基本は $GOPATH の下__ で開発する

```
($GOPATH = $HOME/gopath の例)

$GOPATH [=$HOME/gopath]
bin/
    streak                         # コマンド実行形式ファイル
    todo                           # コマンド実行形式ファイル
pkg/
    linux_amd64/
        code.google.com/p/goauth2/
            oauth.a                # パッケージオブジェクト
        github.com/nf/todo/
            task.a                 # パッケージオブジェクト
src/
    code.google.com/p/goauth2/
        .hg/                       # mercurialレポジトリメタデータ
        oauth/
            oauth.go               # パッケージソース
            oauth_test.go          # テストソース
    github.com/nf/
        streak/
            .git/                  # gitレポジトリメタデータ
            oauth.go               # コマンドソース
            streak.go              # コマンドソース
        todo/
            .git/                  # gitレポジトリメタデータ
            task/
                task.go            # パッケージソース
            todo.go                # コマンドソース
```

#### 環境変数GOROOTとGOPATH
* `GOROOT` - golangインストールルートフォルダ
* `GOPATH` - golangのユーザーパッケージのインストールフォルダ


### パッケージ
* パッケージ ≒ 名前空間, __1つのファイルに1つの名前空間が割り当てられる__
* `package [name]`で宣言
* `import [name]`でパッケージをインポート

```go
import f "fmt"  // fmtパッケージをfという名前でインポート, f.Printf("abc") という使い方

import "fmt"  // fmtパッケージをそのままの名前でインポート, fmt.Printf("abc") という使い方

import (
  "fmt"  // 複数ある場合は () でグルーピング
  "os"
  "github.com/user/project"  // リポジトリサービスのURL指定, githubとかcode.google.comとか
)

```

* __"自分で作ったパッケージについては、将来追加されるライブラリやその他の外部ライブラリと衝突しないであろうベースパスを選ぶ必要があります。"__
* 実装上、同一パッケージ内のすべてのソースファイルが、同一ディレクトリ内に置かれている必要がある

#### package main と func main() について
* __`main`package の `main` func__


### import path (インポートパス)
* ローカルファイルシステム上でパッケージを参照する際のパス
    * ファイルシステム風にパッケージを参照できるということは、例えば相対パス参照なんかも可能
* 各goコマンドで引数として`[packages]`を取る場合、このimport path仕様に則った形式で指定する
* ドット(`.`)の意味
    * `.` - カレントディレクトリ(と、その直下のパッケージ)
    * `..` - 親ディレクトリ(と、その直下のパッケージ)
    * `...` - 全てのサブディレクトリ（と、その直下のパッケージ）
        * `x/...` - xとその全てのサブディレクトリ・パッケージ を指す
* `go help importpath`で詳細確認可能

* 例 : あるディレクトリ上にルートパッケージが`hoge`のgoプロジェクトがあり、その配下で`huga`というディレクトリ・パッケージが存在する場合
    * `hoge/huga`パッケージは`./huga`で参照可能
        * 逆にhugaサブディレクトリから親であるhogeパッケージは`..`で参照可能
    * `./...` で全てのサブパッケージを参照
* リモートのパスも参照可能で、下記ホスティングサービスをサポートしている
    * BitBucket(Git, Mercurial) - `bitbucket.org/user/project` `bitbucket.org/user/project/sub/directory`
    * Github(Git) - `github.com/user/project` `github.com/user/project/sub/directory`
    * Google Code Project Hosting(Git, Mercurial, Subversion) - `code.google.com/p/project` `code.google.com/p/project/sub/directory`
    * Launchpad(Bazaar) - `launchpad.net/project` `launchpad.net/project/sub/directory`
* ソース内で`import "./subpkg"`的な指定をする場合の注意点 - [ログ取得ツール » Blog Archive » golangの欠陥](http://wtnb.mydns.jp/wordpress/archives/6149)
    * __`$GOPATH/src/PJT`上で開発してる場合、上記のような相対パスimportは"local import ... non-local package"エラーでコケる__
    * つまり`github.com/goldeneggg/hoge`と書かないとイケない
    * これで何が問題かというと、 __`go get`で常にリモートのmaster（のHEAD）を持ってこようとする__ 事になる
        * __ローカルは新しい、リモートはまだ古いまま、というケースでコンパイルが通らなくなる__

### プログラムなのか？ライブラリなのか？それぞれに適したワークスペース構成とは？
___利用者が`go get`や`go install`した時に、どういう挙動になるのか？彼等の意図に沿った挙動を実現できているのか？を考えて開発する事が大事___


* __プロジェクトがプログラムorライブラリ いずれを担っているのかちゃんと考えるべき__
    * `github.com/goldeneggg/cat` - プログラムっぽい
        * installして利用されるという想定で
        * ROOTDIRにmain package（ のmain() ) を置くのが王道。build実行DIRもココ
            * `app`とか`cmd`ってサブDIR掘って、ココに置くってケースもあるかも知れない
                * build時にpackage pathとして`go build ./{app,cmd}`って指定する
            * "ROOTDIR 整頓シンドローム"
            * ___でもコレってリモートからの`go get`もちゃんと動くんだっけ？___
                * `go get`が`$GOPATH/src`以下にソースをcloneしてるのでつまり。。。 __buildでコケる__

                ```
                $ go get github.com/goldeneggg/ipcl
                package github.com/goldeneggg/ipcl
                        imports github.com/goldeneggg/ipcl
                        imports github.com/goldeneggg/ipcl: no buildable Go source files in /home/ahouser/gopath/src/github.com/goldeneggg/ipcl
                ```

            * __こういう場合に"`go get`でのインストールを非推奨扱いとする" というスタンスのプロジェクトもありそう__
                * pecoとかそんな感じに見える
                * もしpecoコマンドを`go get`で入れたければ、`go get github.com/peco/peco/cmd/peco`とすれば良い
                    * __`cmd/peco`とcmdの下にコマンド名と同じpackageを挟んでいるのがキモ__ これやらないと`go get`でインストールされるpecoコマンドの実体が`cmd`って名前になってしまう
    * `github.com/goldeneggg/libhoge` - ライブラリっぽい
        * 他プロジェクトでimport利用されるという想定で
        * __ROOTDIRに"libhoge"って名前のpackage・ソースがずらずらと配置されてるのが一番シンプルな構成__
* pecoが何故にあのようなDIR/package構成になっているのか、少し意図が理解できた気がする
    * pecoは __ルートディレクトリに配置したソースを`peco`というpackageに所属させて、mainのソースは`ROOT/cmd/peco`に配置してる
    * Goでよく見かけるパターン(＊)とは異なる
        * ルートディレクトリにmainを(main.goみたいな名前で)配置
        * package名と同じDIRを作って、その配下のソースはDIR名のpackageに所属させる
        * ってドキュメントでも推奨されてる
    * そのpackageはツール(cli app)なのか、ライブラリなのか、両方提供してるのか、を意識する事が大事
    * ライブラリなら、他プロジェクトからimportされて利用されるかもしれない
    * その際のimport文は`import github.com/peco/peco`となる
    * 仮に(＊)のパターンに沿った構成にしている場合、例えばimport文が`import github.com/peco/peco/lib`といった具合に、サブpackageを指定してimportする事になりそう
        * どちらが直感的に分かりやすいか？ => `peco/peco`の方が分かりやすいよね


#### 整理



#### 公開されてるツールに見る、ワークスペースとパッケージの構成例



### vim環境整備

#### 公式のツールを使う

* `$GOROOT/misc/vim`にvim用の設定ファイル一式があるので、これを導入
    * `$GOROOT/misc/vim/readme.txt`を参照

```vim
set runtimepath+=$GOROOT/misc/vim

autocmd FileType go autocmd BufWritePre <buffer> Fmt
```

* __golangの標準インデントは タブ らしい__

#### プラグインを使う
* [Go development environment for Vim - The Gopher Academy Blog](http://blog.gopheracademy.com/vimgo-development-environment) がヨサゲ
* TODO あとで試す
