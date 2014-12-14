## 主なコマンド
とりあえず下記を読んどくべし

* [Command Documentation - The Go Programming Language](https://golang.org/doc/cmd)
    * [日本語訳](http://golang-jp.org/cmd/)


```
Go is a tool for managing Go source code.

Usage:

        go command [arguments]

The commands are:

    build       compile packages and dependencies
    clean       remove object files
    env         print Go environment information
    fix         run go tool fix on packages
    fmt         run gofmt on package sources
    get         download and install packages and dependencies
    install     compile and install packages and dependencies
    list        list packages
    run         compile and run Go program
    test        test packages
    tool        run specified go tool
    version     print Go version
    vet         run go tool vet on packages

Use "go help [command]" for more information about a command.

Additional help topics:

    c           calling between Go and C
    gopath      GOPATH environment variable
    importpath  import path syntax
    packages    description of package lists
    testflag    description of testing flags
    testfunc    description of testing functions

Use "go help [topic]" for more information about that topic.
```

### go build
* `go build [packages|name|(nothing)]`で、指定したパッケージ, ソースファイル, ディレクトリの内容をビルドしてバイナリを出力する

### go install
* `go install [packages]`で、指定したパッケージを`$GOPATH`にダウンロード？＆ビルドし、バイナリをインストールする
* `$GOPATH/pkg/PACKAGE PATH`にビルド結果のバイナリ(`.a`ファイルとか)が出力される
    * ライブラリの場合は、これでPJTからのimportが可能になる
* プログラムは`$GOPATH/bin`にインストールされる__ (ここが`go build`との違い)

### go get
* `go get [packages]`で、指定したパッケージを`$GOPATH`にインストールする
* `[packages]`で指定したパッケージ __と、それが依存している(使用している)パッケージ__ をインストールする
* 自分で開発中のプロジェクトで必要な外部パッケージを取得したい場合に使う
    * `-d`オプション - ダウンロードのみ行いインストールは行わない場合
        * `go get -d ./...` で、 __ルートパッケージ＆全サブパッケージの依存するもの全てのインストール__ が行われる
    * `-u`オプション - インストール済のパッケージの更新を行いたい場合

### go run
* `go run [build flags] [-exec xprog] gofiles... [arguments...]` goファイルを実行する
* 単体のgoファイルを実行する場合、そのgoファイルがmainパッケージで且つmain()関数が定義されていること
    * main()関数から参照されている変数・関数が、同じmainパッケージの別ファイルに定義されている場合は、`go run`の実行ファイルを複数指定する


### go env
* `go env`で、goに関する環境変数一覧を表示する

### go list
* `go list`で、`package`や`import`で指定したパッケージの一覧を表示する
    * `-e` - エラーが有る場合、そのエラー情報を表示
    * `-f` - 結果をどうのように表示するかを、[templateパッケージ](https://golang.org/pkg/text/template/)のルールに準拠した形式で指定する。デフォルトは`'{{.ImportPath}}'`
        * `go list -f '{{.ImportPath}},{{join .Imports ","}}'` - 自packageとimportしているパッケージをカンマ区切りで表示する
        * `go list -f '{{.ImportPath}}{{range .Imports}},{{.}}{{end}}'` - 上記と同じ結果。`range...end`は配列やmapのループで使う関数で、ループ内要素は`.`で参照可能
        * `go list all` - ローカルファイルシステム上の`GOPATH`ツリー配下に存在する全パッケージを一覧表示する
        * 主な指定可能項目
            * `ImportPath string`
            * `Name string`
            * `GoFiles []string`
            * `Imports []string`
            * `Incomplete bool`

### go tool
* `go tool TOOLCMD`でツールコマンドを実行する, 引数なしの場合、実行可能なTOOLCMDを確認出来る

```
6a
6c
6g
6l
addr2line
cgo
cover
dist
fix
nm
objdump
pack
pprof
tour
vet
yacc
```

#### go tool fix
* Goのプログラムに古いAPIが使われていないかを探し、新しいAPIに書き換えます。 Goのリリースによって、あなたのプログラムに修正の必要が生じたときにfixは役立ちます。


#### go tool pprof
* __多機能優秀プロファイラ__

```
Usage:
pprof [options] <program> <profiles>
   <profiles> is a space separated list of profile names.
pprof [options] <symbolized-profiles>
   <symbolized-profiles> is a list of profile files where each file contains
   the necessary symbol mappings  as well as profile data (likely generated
   with --raw).
pprof [options] <profile>
   <profile> is a remote form.  Symbols are obtained from host:port/pprof/symbol

   Each name can be:
   /path/to/profile        - a path to a profile file
   host:port[/<service>]   - a location of a service to get profile from

   The /<service> can be /pprof/heap, /pprof/profile, /pprof/pmuprofile,
                         /pprof/growth, /pprof/contention, /pprof/wall,
                         /pprof/thread, /pprof/block or /pprof/filteredprofile.
   For instance:
     pprof http://myserver.com:80/pprof/heap
   If /<service> is omitted, the service defaults to /pprof/profile (cpu profiling).
pprof --symbols <program>
   Maps addresses to symbol names.  In this mode, stdin should be a
   list of library mappings, in the same format as is found in the heap-
   and cpu-profile files (this loosely matches that of /proc/self/maps
   on linux), followed by a list of hex addresses to map, one per line.

   For more help with querying remote servers, including how to add the
   necessary server-side support code, see this filename (or one like it):

   /usr/doc/google-perftools-1.5/pprof_remote_servers.html

Options:
   --cum               Sort by cumulative data
   --base=<base>       Subtract <base> from <profile> before display
   --interactive       Run in interactive mode (interactive "help" gives help) [default]
   --seconds=<n>       Length of time for dynamic profiles [default=30 secs]
   --add_lib=<file>    Read additional symbols and line info from the given library
   --lib_prefix=<dir>  Comma separated list of library path prefixes

Reporting Granularity:
   --addresses         Report at address level
   --lines             Report at source line level
   --functions         Report at function level [default]
   --files             Report at source file level

Output type:
   --text              Generate text report
   --callgrind         Generate callgrind format to stdout
   --gv                Generate Postscript and display
   --web               Generate SVG and display
   --list=<regexp>     Generate source listing of matching routines
   --disasm=<regexp>   Generate disassembly of matching routines
   --symbols           Print demangled symbol names found at given addresses
   --dot               Generate DOT file to stdout
   --ps                Generate Postcript to stdout
   --pdf               Generate PDF to stdout
   --svg               Generate SVG to stdout
   --gif               Generate GIF to stdout
   --raw               Generate symbolized pprof data (useful with remote fetch)

Heap-Profile Options:
   --inuse_space       Display in-use (mega)bytes [default]
   --inuse_objects     Display in-use objects
   --alloc_space       Display allocated (mega)bytes
   --alloc_objects     Display allocated objects
   --show_bytes        Display space in bytes
   --drop_negative     Ignore negative differences

Contention-profile options:
   --total_delay       Display total delay at each region [default]
   --contentions       Display number of delays at each region
   --mean_delay        Display mean delay at each region

Call-graph Options:
   --nodecount=<n>     Show at most so many nodes [default=80]
   --nodefraction=<f>  Hide nodes below <f>*total [default=.005]
   --edgefraction=<f>  Hide edges below <f>*total [default=.001]
   --focus=<regexp>    Focus on nodes matching <regexp>
   --ignore=<regexp>   Ignore nodes matching <regexp>
   --scale=<n>         Set GV scaling [default=0]
   --heapcheck         Make nodes with non-0 object counts
                       (i.e. direct leak generators) more visible

Miscellaneous:
   --tools=<prefix>    Prefix for object tool pathnames
   --test              Run unit tests
   --help              This message
   --version           Version information

Environment Variables:
   PPROF_TMPDIR        Profiles directory. Defaults to $HOME/pprof
   PPROF_TOOLS         Prefix for object tools pathnames

Examples:

pprof /bin/ls ls.prof
                       Enters "interactive" mode
pprof --text /bin/ls ls.prof
                       Outputs one line per procedure
pprof --web /bin/ls ls.prof
                       Displays annotated call-graph in web browser
pprof --gv /bin/ls ls.prof
                       Displays annotated call-graph via 'gv'
pprof --gv --focus=Mutex /bin/ls ls.prof
                       Restricts to code paths including a .*Mutex.* entry
pprof --gv --focus=Mutex --ignore=string /bin/ls ls.prof
                       Code paths including Mutex but not string
pprof --list=getdir /bin/ls ls.prof
                       (Per-line) annotated source listing for getdir()
pprof --disasm=getdir /bin/ls ls.prof
                       (Per-PC) annotated disassembly for getdir()

pprof http://localhost:1234/
                       Enters "interactive" mode
pprof --text localhost:1234
                       Outputs one line per procedure for localhost:1234
pprof --raw localhost:1234 > ./local.raw
pprof --text ./local.raw
                       Fetches a remote profile for later analysis and then
                       analyzes it in text mode.
```

* `runtime/pprof`パッケージと`net/http/pprof`パッケージの使い方を合わせてマスターしてプロファイリングに活用する
    * [golang の net/http/pprof を触ってみたメモ - sonots:blog](http://blog.livedoor.jp/sonots/archives/39879160.html)

##### 使い方ざっくり(cmd編)
cpu profileの例  ___macだと動かないって___

* プロファイリングしたいソースに下記を埋め込む

```go
func main() {
	// cpu profile
	f, err := os.OpenFile("cpuprof.log", os.O_WRONLY|os.O_CREATE, 0755)
	if err != nil {
		fmt.Println("Error file open", err)
		return
	}
	pprof.StartCPUProfile(f)
	defer pprof.StopCPUProfile()

  // :
  // 実処理
  // :
}
```

* `go build PROGRAM`する, 実行ファイルを出力
* `./PROGRAM`で実行, プロファイル結果ファイルが出力される
* `go tool pprof PROGRAM PROFILE`でプロファイラ起動

```
Welcome to pprof!  For help, type 'help'.
(pprof)
```

* プロファイラ内での主なコマンド
    * `topN`
    * `web`

##### 使い方ざっくり(net/http/pprof編)
[pprof - The Go Programming Language](http://golang.org/pkg/net/http/pprof/)

[golangで書かれたプログラムのメモリ使用状況を見る - はこべブログ ♨](http://hakobe932.hatenablog.com/entry/2014/04/10/010619)

* 対象ソースにこんな感じのコードを追加する

```go
import _ "net/http/pprof"

go func() {
  log.Println("localhost:6060", nil)
}()
```

* このプログラムをビルド・実行すると、 __その実行中に__ ブラウザから`http://localhost:6060/debug/pprof`にアクセスしてプロファイル結果が閲覧できる
* 加えて、この実行中に `go tool pprof http://localhost:6060/debug/pprof/XXX`することでcliのpprofを起動してプロファイル情報を確認することが出来る, XXX は下記
    * `heap`
    * `profile`
    * `block`

