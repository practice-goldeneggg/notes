## 公開されているツール/ライブラリを使う
rubyやnodejsほどライブラリ検索・導入の仕組みが充実しているわけでは（まだ）無いが、外部ツールやライブラリを探す/調べる際に有用な情報はこちら

* [Go Search - Find popular and relevant Go packages!](http://go-search.org/)
* [GoDoc](http://godoc.org/)
* [Projects - go-wiki - A list of Go projects. - Go Language Community Wiki - Google Project Hosting](https://code.google.com/p/go-wiki/wiki/Projects)


### インストール
* `go get [パッケージ名]`でインストールして使う
    * `go get code.google.com/p/...`
    * `go get github.com/...`

* [codegangsta/negroni](https://github.com/codegangsta/negroni) を試してみる

```
% go get github.com/codegangsta/negroni  # $GOPATH/ 下にインストールされる

% vi negroni_server.go
```
```go
package main

import (
	"fmt"
	"github.com/codegangsta/negroni"  // go get したパッケージをインポート
	"net/http"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
		fmt.Fprintf(w, "Welcome to the home page!")
	})

	n := negroni.Classic()
	n.UseHandler(mux)
	n.Run(":4000")
}
```
```
% go run negroni_server.go
[negroni] listening on :4000
```

* http://localhost:4000/ にアクセスして "Welcome to the home page!" が見えたらOK

```
[negroni] Started GET /
[negroni] Completed 200 OK in 65.967us
[negroni] Started GET /favicon.ico
```

* 導入したパッケージは`go list`で確認可能
    * `./...`は "カレントディレクトリの" "全てのパッケージ" を意味する

```
% cd $GOPATH
% go list ./...
github.com/codegangsta/negroni
```

### ツール

#### gocode
* コード補完ツール
* `go get github.com/nsf/gocode`でインストール


#### peco
[Big Sky :: Windows のコマンドプロンプトを10倍便利にするコマンド「peco」](http://mattn.kaoriya.net/software/peco.htm)

```
% go get github.com/peco/peco/cmd/peco
```

* 使い方 - インタラクティブに実行したいコマンドを`COMMAND | peco`と叩く
    * 例えば`cd $(ls | peco)`とすると、移動先を選択してcd
* 設定ファイルによる挙動のカスタマイズが可能
    * JSONで`$HOME/.peco/config.json`に配置 (自動で4つのDIRが探索されるが、何処にもない場合に左記DIRのjsonが読み込まれる)
    * `Keymap`というkey項目配下に `"key" : "action"`形式で定義する
    * `Style`というkey項目配下に `"key" : "action"`形式で定義する

    ```json
    {
      "keymap" : {
        "C-p" : "peco.SelectPrevious"
        "C-n" : "peco.SelectNext"
      }
    }

    {
      "Style": {
        "Basic": ["on_default", "default"],
        "Selected": ["underline", "on_cyan", "black"],
        "Query": ["yellow", "bold"]
      }
    }
    ```


#### groupcache
[groupcache を読む #1 - Kato Kazuyoshi](http://2013.8-p.info/japanese/07-31-groupcache-1.html)

```
go get github.com/golang/groupcache
```

* Goで書かれたキャッシュライブラリ。名前の末尾に"d"がつかないことからもわかるように、memcached のようなデーモンではない
* dl.google.com で使用されている


### ライブラリ

#### go-flags
[Go言語でコマンドラインオプション使うなら。便利パッケージgo-flags。 - Thinking-megane](http://blog.monochromegane.com/blog/2014/01/23/go-flags/)

```
go get github.com/jessevdk/go-flags
```

* 標準パッケージの`flag`よりも便利なコマンドラインオプションパーサー
* 構造体でオプションの仕様を定義
    * __メンバーの変数名はUppercase(public)にすること__
* `flags.NewParser(a1, a2)`する時、a2には`Option`型の定数を設定するが、`Default`は`HelpFlag | PrintErrors | PassDoubleDash`とconst定義されている
    * この`HelpFlag`は`-h``--help`を付けて実行した際に自動でヘルプメッセージを表示する機能を有効にしてくれるのだが、`Parse()`した際にerror(中身はヘルプ文言)も一緒に返してくる
    * つまり __`Parse()`で予期せぬエラーが発生したのか、それともヘルプ表示を実行されたのか__ という区別が `if err != nil`な判定では行えない


#### codegangsta/cli
[codegangsta/cli](https://github.com/codegangsta/cli)

```
go get github.com/codegangsta/cli
```

* `Context`,`App`,`Flag`,`Command`型の概念を理解する
    * cliアプリケーションの情報を表す`App`
    * (サブ)コマンドの情報を表す`Command`
        * gitコマンドが良い例(`git commit`とか`git checkout`とかの commit,checkout をCommandで表す)
    * オプション引数の情報を表す`Flag`
    * 実行時の環境・情報について保持する`Context`
* Flagの設定はCommand単位, App単位(=Global)の2種類
    * 実行時は`APP_NAME <global options> COMMAND_NAME <command options>`の形式で指定。順番に注意

```go
type Context struct {
	App       *App
	Command   Command
	flagSet   *flag.FlagSet
	globalSet *flag.FlagSet
	setFlags  map[string]bool
}

type App struct {
	// The name of the program. Defaults to os.Args[0]
	Name string
	// Description of the program.
	Usage string
	// Version of the program
	Version string
	// List of commands to execute
	Commands []Command
	// List of flags to parse
	Flags []Flag
	// Boolean to enable bash completion commands
	EnableBashCompletion bool
	// Boolean to hide built-in help command
	HideHelp bool
	// An action to execute when the bash-completion flag is set
	BashComplete func(context *Context)
	// An action to execute before any subcommands are run, but after the context is ready
	// If a non-nil error is returned, no subcommands are run
	Before func(context *Context) error
	// The action to execute when no subcommands are specified
	Action func(context *Context)
	// Execute this function if the proper command cannot be found
	CommandNotFound func(context *Context, command string)
	// Compilation date
	Compiled time.Time
	// Author
	Author string
	// Author e-mail
	Email string
}

type Command struct {
	// The name of the command
	Name string
	// short name of the command. Typically one character
	ShortName string
	// A short description of the usage of this command
	Usage string
	// A longer explanation of how the command works
	Description string
	// The function to call when checking for bash command completions
	BashComplete func(context *Context)
	// An action to execute before any sub-subcommands are run, but after the context is ready
	// If a non-nil error is returned, no sub-subcommands are run
	Before func(context *Context) error
	// The function to call when this command is invoked
	Action func(context *Context)
	// List of child commands
	Subcommands []Command
	// List of flags to parse
	Flags []Flag
	// Treat all flags as normal arguments if true
	SkipFlagParsing bool
	// Boolean to hide built-in help command
	HideHelp bool
}

type Flag interface {
	fmt.Stringer
	// Apply Flag settings to the given flag set
	Apply(*flag.FlagSet)
	getName() string
}
```


#### goquery

* JQueryライクなセレクタを持つHTTPリクエスト/スクレイピング用ライブラリ

```
% go get github.com/PuerkitoBio/goquery
```

##### cascadia

* goquery内でも使用しているCSSセレクタ解析ライブラリ

```
% go get code.google.com/p/cascadia
```

#### godep

* 依存関係管理ツール

```
% go get github.com/tools/godep
```

* 管理したい自プロジェクトのdirに移動して`godep save`する
    * 自プロジェクトに複数パッケージが存在する場合は、それぞれの内部で使用している依存ライブラリをインストールしたい => `godep save ./...`とする

```
% cd [target dir]
% godep save
または
$ godep save ./...
```

* するとソースのimportを解析した結果が`Godeps`というディレクトリに出力される

```
% ls -l Godeps
Godeps.json     Readme          _workspace


% cat Godeps.json
{
  "ImportPath": "github.com/goldeneggg/ipcl",
  "GoVersion": "go1.3",
  "Deps": [
    {
      "ImportPath": "github.com/jessevdk/go-flags",
      "Comment": "v1-200-g7047cf7",
      "Rev": "7047cf7a8dc6f41e53365420ab62d415055232c6"
    }
  ]
}
```

* `godep restore`すると、依存関係のあるライブラリが`$GOPATH`にインストールされる
* godep導入後の作業の流れ
    * 開発 - テスト のサイクル
        * ソースを修正する
        * `godep go test [test target]`する。テストと一緒に依存ライブラリの最新化が動く
    * 依存パッケージの追加
        * `go get [package]`する
        * 対象ソースに`import`文を追加する
        * `godep save`する
    * 依存パッケージのアップデート
        * `go get -u [package]`する
        * `godep update [package]`する
    * __goコマンドの前に`godep`をつけると、`_workspace`ディレクトリにあるパッケージ（ソース）を使うようになる__. `godep go build`とか

#### go.tools

* 本体とは分離して "code.google.com/p/go.tools" で管理されているツールライブラリ
* `go get [ツール(のURL)]` でインストールして使う

```
% go get code.google.com/p/go.tools/cmd/[ツール名]
```

##### goimports
* ファイル保存時に自動フォーマット + 使ってないimportの削除 までやってくれる
    * gofmtはフォーマットまで

```
% go get code.google.com/p/go.tools/cmd/goimports
```

* vimの設定, fmt使ってた部分を`goimports`にすればOK



#### go.net
* [go.net](https://code.google.com/p/go/source/browse/?repo=net) で提供されている追加ライブラリ群

##### context
* [Go Concurrency Patterns: Context - The Go Blog](http://blog.golang.org/context)
* Goのサーバではリクエストはサーバ自体のゴルーチンにハンドリングされる
* リクエストハンドラはDB等のバックエンドサービスにアクセスするためのゴルーチンを起動することがよくある
* こうして起動されたゴルーチンではリクエスト内に含まれる情報（ユーザーデータや認証トークンのような）にアクセスする必要がある(ことが多い)
* リクエストがキャンセルされたりタイムアウトした場合は、これら起動されたゴルーチンも速やかにexitするべき -> そうすればリソースの節約・有効活用に繋がる
* `context`パッケージは、こうした __ハンドリングされたリクエストを起点に起動したゴルーチンとメイン・ゴルーチン間でリクエスト内容・キャンセル通知・APIをまたいだデッドライン管理 を容易にする為のもの


##### spdy


##### websocket



