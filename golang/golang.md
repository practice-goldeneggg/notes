# golang


## ブクマ
* [The Go Programming Language](http://golang.org/)
    * [翻訳版(途中)](http://golang-jp.org/)
* [golang.jp - プログラミング言語Goの情報サイト](http://golang.jp/)
* [Projects - go-wiki - A list of Go projects. - Go Language Community Wiki - Google Project Hosting](https://code.google.com/p/go-wiki/wiki/Projects)
* [Go言語の初心者が見ると幸せになれる場所 - Qiita](http://qiita.com/tenntenn/items/0e33a4959250d1a55045)
* [Go by Example](https://gobyexample.com/)
* [Go Search - Find popular and relevant Go packages!](http://go-search.org/)
* [GoDoc](http://godoc.org/)


## GO言語とは
* Google製、オープンソースのプログラミング言語
    * BSDライセンス
* プロセッサを複数サポート、x86, amd64, ARMで動く
* Linux、Mac、Native Clientで動作する開発言語で、 __Android携帯上でも動作__
    * Windowsでは動かない
* シンプル, コンパイル・実行速度が速い, 安全性高い, 同期処理が容易
* コンパイル言語である
    * 「gc Go言語コンパイラ」、もうひとつはGNUコンパイラコレクション(GCC)の一部である「gccgoコンパイラ」 コンパイラはこの2つ
        * gcコンパイラの方がより完成されていてテストも十分にされている, __速い__
* 型安全, メモリセーフ
    * 型がある
    * "一見、動的言語のようですが実行速度と安全性は静的言語そのものです"
* ポインタの概念有り, ただしCのような演算は不可
* `goroutine`と名づけられた軽量 __通信プロセス__ により、サーバ処理が書きやすくなっている
* `map`がコレクションライブラリではなく __組込の機能として__ 提供されている

### C++との違い
* クラスメソッド、継承によるクラスの階層、仮想関数が無い
    * 代わりとして、インタフェースが用意されている
* ガベージコレクションを採用している
* 配列が値である
    * 配列が関数パラメータとして使用されるときには、関数はその配列へのポインタではなく、コピーを受け取る

#### メモリの確保について
* __C/C++においては、動的なメモリ確保はその都度OSに問い合せることで行われ、非常に大きなコストとなる__
* このため、チャンク(まとめて)確保をしてしまい管理は自分でするというような手法を用いる(独自アロケータなど)
* Goも例外ではなく、 __例えばスライスについては、都度appendするような処理が走る場合、予めcapacityを見積もっておいてその分の初期capacityを確保しておくのが望ましい__


## インストール
[Getting Started - The Go Programming Language](http://golang.org/doc/install)

### Mac
* [Downloads - go - Binary and source download locations. - The Go Programming Language - Google Project Hosting](https://code.google.com/p/go/wiki/Downloads?tm=2) からpkgを落として入れる
* `GOROOT`環境変数に`/usr/local/go`を追加する
    * インストール先を変更する場合は`GOROOT`の設定値も変える
* `GOPATH`環境変数に`$HOME/gopath`(例)を追加する
    * dirが無ければ作る
* `PATH`環境変数に`$GOROOT/bin``$GOPATH/bin`を追加する
* サンプルソースで動作確認

```go
// +build main1   // <= [TODO] なんか同じディレクトリに package main, func main() を含んでるソースを2つ置いたらコンパイルエラーになったのでこれ書いて回避してる

package main

import "fmt"

func main() {
  fmt.Printf("hello world\n")
}
```
```
% go run hello.go
hello world
```

### 環境変数GOROOTとGOPATH
* `GOROOT` - golangインストールルートフォルダ
* `GOPATH` - golangのユーザーパッケージのインストールフォルダ


## 開発時のディレクトリ構成
* [golang - 【翻訳】プロダクション環境でのベストプラクティス - Qiita](http://qiita.com/umisama/items/c2a8db6c23db18dd5437) の開発環境についての節を参照
* __基本は $GOPATH の下__ で開発する

```
($GOPATH = $HOME/gopath の例)

$GOPATH [=$HOME/gopath]
├── pkg // go getしたパッケージ(のbuild成果物)はこの下
│   └── darwin_amd64
│       └── github.com
│           └── codegangsta // go getでインストールしたパッケージ のbuild成果物
└── src // go getしたパッケージのソースはこの下。 & 自分の開発もこの下で
    ├── github.com
    │   ├── codegangsta
    │   │   └── negroni // go getでインストールしたパッケージのソース
    │   └── jpshadowapps // 自分のリポジトリルート
    │       ├ipcl
    │       └── golang_first
```



## 主なコマンド

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

### go get
* `go get [packages]`で、指定したパッケージを`$GOPATH`にインストールする
* `[packages]`で指定したパッケージ __と、それが依存している(使用している)パッケージ__ をインストールする
* 自分で開発中のプロジェクトで必要な外部パッケージを取得したい場合に使う
    * `-d`オプション - ダウンロードのみ行いインストールは行わない場合
        * `go get -d ./...` で、 __ルートパッケージ＆全サブパッケージの依存するもの全てのインストール__ が行われる
    * `-u`オプション - インストール済のパッケージの更新を行いたい場合

### go build
* `go build [packages|name|(nothing)]`で、指定したパッケージ, ソースファイル, ディレクトリの内容をビルドしてバイナリを出力する

### go install
* `go install [packages]`で、指定したパッケージを`$GOPATH`にダウンロード？＆ビルドし、バイナリをインストールする
* `$GOPATH/bin`にインストールされる

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


## 主な言語仕様

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

import . "fmt"  // ドットインポート : パッケージがインポートされた後このパッケージの関数をコールする際、パッケージ名を省略することができる 事を意味する(=このソース自信が パッケージfmt であるかのように振る舞える)
```

* ドットインポートの用途については  を参照
    * `hoge`パッケージのテストを`hoge_test`パッケージにし、`hoge`の機能をパッケージ名無しで呼び出したい場合
        * このtipsは __循環参照回避策としても使える__
            * パッケージ foo のテストファイルが “bar/testutil” をインポートしていて、”bar/testutil” が “foo” をインポートするような場合、テストファイルをパッケージ foo に含めると循
* __実装上、同一パッケージ内のすべてのソースファイルが、同一ディレクトリ内に置かれている必要がある__

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



### ビルトイン関数
[builtin - The Go Programming Language](http://golang.org/pkg/builtin/), 詳細は各サブ節で書くとして、どんなビルトイン関数があるかは左記URLを参照


### 宣言

#### 変数
* `var hoge string = ""` - 変数名, 型, 初期値
* `var hoge = ""` - 変数名, 初期値
* `hoge := ""` 変数名, 初期値。 `:=`が __初期化__ を意味する。 __つまり同じ変数に対して`:=`を複数回使用したらコンパイルエラーになる__

#### 定数, iota
* `const Pi float64 = 3.14159265358979323846` - 1行で定数を宣言する。名前と型と値を指定

* `()`で括って定数をリストで宣言する, __リスト内では先頭の宣言を除き、式リスト/型は省略可能(直近で使用した式リストが適用され、前の式リストを繰り返すことと同じになる)__
* `iota`は "事前宣言済み識別子" で、定数の宣言で使用する
    * __予約語constがソース内に現れると値が0にリセットされ、各ConstSpecのあとで値がひとつ増やされる__

```go
const (
  Sunday = iota
  Monday
  Tuesday
  Wednesday
  Thursday
  Friday
  Partyday
  numberOfDays  // この定数はエクスポートされない
)
```


### 配列
* goの配列は __値__ である
    * cのように "先頭の配列要素へのポインタ" というわけではない
* 配列変数
    * `var hoge[3]int`(サイズが3)
    * `var hoge [3]int{1,2,3}`(サイズが3で要素が固定)
* 配列リテラル
    * `[2]int{1,2}`(要素数＋要素)
    * `[...]int{1, 2}`(要素数を数えず ... で省略)  __何も書かない場合との違いは？ => 何も書かないとスライス__


### 文字列
* `string`型
* 文字列型の要素はbyte型データを持っているため、一般的なインデックスを使ったアクセスが可能
* `len(strvar)`関数で文字列長を調べることが出来る
* 文字列に対して`range`のループを記述すると、 __UTF-8エンコードでUnicodeの各文字を取り出す__ `for pos, char:= range "日本語" { ... }`
* リテラル
    * 未加工文字列リテラル``hoge`` - バッククォートで囲んだ文字列リテラル, バックスラッシュや改行もそのまま含まれる
    * 解釈有文字列リテラル`"hoge"` - ダブルクォートで囲んだ文字列リテラル, バックスラッシュはエスケープ用、や改行を含めることは出来ない


### スライス
[SliceTricks - go-wiki - How to do vector-esque things with slices. - Go Language Community Wiki - Google Project Hosting](https://code.google.com/p/go-wiki/wiki/SliceTricks)

* スライスは配列と似ているが、
    * サイズを明示的に持たず([]と[10]の違い)
    * 作成元である通常の配列(匿名であることが多い)を部分的に参照している
    * 配列だけではデータを共有はできないが、 __複数のスライスが同一の配列を参照することでデータ共有が可能になる__
* __どのような配列・サイズであっても要素型が同じであれば、それを参照するスライス変数を宣言できる__
    * __スライスは実体への参照である__
* 通常の配列より柔軟で、実体への参照であり、かつ効果的であるためGo言語のプログラム内で多用される
* 関数に配列を受け渡すときは、大抵はパラメータをスライスとして宣言する
    * "関数を呼び出すとき、効率のため配列をスライスし、スライス参照を作成し、それを渡すようにしてください"
* スライス変数 - `var hoge []int`
* スライスリテラル - `[]int{1, 2}`(要素のみ)
    * スライスを引数に取る関数を呼ぶ際はこの書き方で`funcname([]int{1, 2})`と呼ぶのが慣例
* スライス式は `a[low : heigh]`形式

#### append関数
* スライスの末尾に要素を追加する
* スライスにスライスを追加する場合は`append(sliceA, sliceB...)`と`...`を付ける。`...`を記述しないと、型が一致しないためコンパイルできない

### メモリ(変数)の割り当て, new / make
* int, 構造体, 配列 - __オブジェクトの内容をコピー__
    * `new`を使う
    * __`new(T)`は`*T`(T型のポインタ)を返す__
        * 割り当てたメモリへのポインタを返す
    * newは指定した方に __ヒープを割り当てる__
    * newによる初期化を __ゼロ化__ と呼ぶ
* map, スライス, channel - __他のオブジェクトへの参照__
    * `make`関数を使う = __参照の初期化__
        * `make([]int, 10, 100)` = 10個(長さが10), キャパシティが100で、int型を持つ配列(への参照)
    * __`make(T)`は`T`(値)を返す__
    * __マップを使用するには最初にmakeを使用して参照を初期化するか、またはすでに作成済みのマップを代入しなければならない__

### 構造体
* 構造体型の定義 - `type Hoge struct { ... }`, 変数の定義 - `var Hoge struct { ... }`

```go
type Point3D struct { x, y, z float64 }
type Line struct { p, q Point3D }
```

* フィールドの宣言時にオプションで文字列リテラルを使って __タグ__ を指定することが出来る
    * タグはリフレクションインタフェースを使って参照可能
        * [unsafeパッケージ - golang.jp](http://golang.jp/go_spec#Package_unsafe)
    * __`encoding/{json,xml}`パッケージで項目と構造体のフィールドをマッピングする際に使われている__
        * jsonの場合、構造体のフィールドタグに``json:"NAME"``と記述されていれば、jsonのマーシャル/アンマーシャル時に`NAME`という項目にマッピングされる
        * 詳細は [json#Marshal - The Go Programming Language](http://golang.org/pkg/encoding/json/#Marshal)を参照

        ```go
        type U struct {
          Alphabet string `json:"alpha"`  // jsonの項目名"alpha"にマッピングされる
        }
        ```

    * __`{text,html}/template`パッケージでHTML内のテンプレート変数と構造体のフィールドをマッピングする際に使われている__
    * __`flags`でのoptionパースでも使われている__

```go
type StringTag struct {
	BoolStr bool   `json:",string"`  // 最後の`...`がタグ
	IntStr  int64  `json:",string"`
	StrStr  string `json:",string"`
}
```

#### コンストラクタと複合リテラル
[コンストラクタと複合リテラル - golang.jp](http://golang.jp/effective_go#composite_literals)

* 複合リテラル - "構造体、配列、スライス、マップを構築し、評価をその都度行い新しい値を作成する"
    * 複合リテラルは __値の型とそれに続く波括弧{}でくくられた要素リスト__ から構成される
* 構造体の初期化(コンストラクタ)においては、 __`hoge := new(Hoge)`と`hoge := &Hoge{}`は等価である, 後者の書き方が 複合リテラル__
    * 複合リテラルではすべてのフィールドを順番通りに記述しなければならない
        * ただし明示的に「フィールド:値」の組み合わせで要素にラベルをつけたときは、イニシャライザは順序通りである必要はない
        * 指定しなかった要素には、その型のゼロ値がセットされる

```
type Point3D struct { x, y, z float64 }

origin := Point3D{}  // Point3Dはゼロ値
origin := &Point3D{y: 1000}  // 複合リテラルのアドレスを取得(§アドレス演算子)すると、リテラル値のインスタンスを指す、ユニークなポインタが生成される)
```

### インタフェース
* __interfaceで定義したメソッドを実装している型であれば（たとえ他のメソッドを有していても）そのinterfaceを実装していることになる__
    * 例えば`type IHoge interface { huga() }`というinterfaceがある場合、`type Hoge struct {...}`のメソッドとして`huga()`を実装すれば、型Hogeはinterface IHoge を実装していることになる
* オブジェクト指向系言語のinterfaceとは勝手が全く違う
    * ラップ, コンポジションがベース
* Reader/Writer あたりのソースが参考になる
* インタフェースに名前をつける際、メソッド名にer(またはr)をつけるというのがGo言語の習わし


### 埋込み
* 要は __構造体の中に構造体（をフィールドとして）記述したり、インタフェースの中にインタフェースを記述することができる__ ということ
    * 「型を埋め込み、実装を借りる」
* インタフェースの埋め込み __インタフェースに埋め込めるのはインタフェースのみ__

```go
type Reader interface {
  Read(p []byte) (n int, err os.Error)
}

type Writer interface {
  Write(p []byte) (n int, err os.Error)
}

type ReaderWriter interface {
  Reader  // Readerが行えること(=Read()メソッド) を組み込む
  Writer  // Writerが行えること(=Write()メソッド) を組み込む
}
```

* 構造体の埋め込み 基本的な考え方はインタフェースと同じ __どんな型でも埋め込める__
    * __フィールド名を付けず匿名で埋め込む__

    ```go
    type Human struct {
      name string
      age int
      weight int
    }

    type Student struct {
      Human  // Human構造体が持っている全てのフィールド・メソッドがStudentにも導入される
      // *Human  // ポインタでも可
      speciality string
    }


    st := Student{Human{"Mike", 15, 60}, "baseball"}  // Humanフィールドは有効なHumanで初期化する必要あり

    fmt.Println(st.speciality)
    fmt.Println(st.name)  // Humanのフィールドも参照可能
    st.methodStudent()  // Studentのメソッドは当然呼べる
    st.methodHuman()  // Humanのメソッドも呼べる, この際のレシーバはStudentではなく埋め込まれたHumanになっている
    ```

    * フィールド名を付けて埋め込むことも可能だが、埋め込んだ型のフィールド・メソッドを参照する際に記述が冗長になる

    ```go
    type Student struct {
      human Human  // human というフィールド名を付けて埋め込み
      speciality string
    }

    st := Student{Human{"Mike", 15, 60}, "baseball"}  // Humanフィールドは有効なHumanで初期化する必要あり

    fmt.Println(st.human.name)  // humanフィールドを経由しての参照
    st.human.methodHuman()  // humanフィールドを経由してのメソッド呼び出し
    ```

* 埋め込み時の名前の競合について。例えば X という名前でフィールド・メソッド名が競合した場合
    * __より深い入れ子内にある X は隠蔽される__
    * __同一階層で X が複数使われている場合はエラーになる__ , ただし __外側のプログラムから参照・使用されなければ問題ない__
    * つまり、 __敢えて競合させて隠蔽(Javaで言うところのオーバーライド)を行う事も可能__


### マップ
* コレクションライブラリではなく組み込みの機能
* `map[KEY_TYPE]VALUE_TYPE`形式で定義
* mapの初期化 `map[string]int{"one":1, "two":2}`  `{KEY:VALUE}`形式で書く
* __カンマOK慣用句 を使うと、mapの値と存在(true/false)を取得可能__ 存在しない場合は値にはゼロ値がセットされる
    * `v, ok = hogeMap[key] // keyが存在すればokはtrue`
    * 値をブランク識別子(`_`)にすると、値を破棄できる。存在チェックだけしたい場合に使える
* mapから値を削除する場合は、 __カンマOK慣用句の右辺左辺を入れ替えればよい__ `hogeMap[key] = 0, false`
* mapに対して`range`のループを記述すると、 __key, valueを個々に取り出せる__ `for k, v := range kv { ... }`

### forループ
* __Goにはwhileは無い。ループはforのみ__
* 複数の変数を回したい時は、CやJavaのように`,`で区切って個別に記述は出来ない。カンマ演算子が無い為
    * 複数の変数を回したい時はこう書く。 複数変数定義時のカンマの使い方と同じで、 __一度に代入するイメージ__ `for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1`


### range
* 要素数のループを繰り返す, 文字列,配列,スライス,map,channel で利用可能
* `for i := 0; i < len(a); i++ { ... }` は `for i, v := range a { ... }`と書ける, __iにはループのインデックスが, vには繰り返し要素が順に代入される__
    * インデックス(key)を使用しない場合はiを省略する, あるいは`for _, v := range a {...}`とプレースホルダを使用して書ける。が、keyを使用しないということを明示する意図で`_`を書くのがgoの慣例？
* 文字列のrangeは __position(key)と文字(value)__ を取り出せる

### 関数
* __関数も値としてとれる, 無名関数も書ける, クロージャ対応もあり__
    * 名前の短い変数に関数を代入して、ショートカットアクセスを実現したり `var p = fmt.Println  // p("hoge") でアクセス`

#### 名前付き結果パラメータ
* __戻り値に名前を付けることができる__
    * __(その型のゼロ値で)初期化が行われた上で、パラメータなしのreturnと結びつけられる__ ので、明確になるだけでなくシンプルになる
* 普通に書くとreturnが色んな箇所に点在するような関数(こうなる設計自体良くないのでは？というツッコミはさておき) なんかに適用すると見通し良くなりそう

```go
func ReadFull(r Reader, buf []byte) (n int, err os.Error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:len(buf)]
    }
    return
}
```

#### 可変長変数リスト
* fmt.Printfの実装は`Printf(format string, v ...interface{}) (n int, errno os.Error)`となっているが、この`...`が可変長変数を表す
    * 可変長変数リストを持つ関数をネスト呼び出しする際は、呼び出す関数に`(v...)`という形式で引数を渡す。こうしないと、vは、単にひとつのスライス引数として渡される


#### init関数
* セットアップのためにパラメータを持たない関数initを定義することができる
    * 初期化中にゴルーチンを起動することはできますが、初期化が完了するまで実行は開始しない。すなわち、初期化処理中に実行されるスレッドは常にひとつだけ
* __インポートしているパッケージの初期化、変数の初期化 が評価された後に呼び出される__


#### 関数リテラル
[関数リテラル - golang.jp](http://golang.jp/go_spec#Function_literals)

* `f := func(x, y int) int { return x + y }` - 変数に代入
* `func(ch chan int) { ch <- ACK }(replyChan)``func() { os.Exit(1) }()` リテラルの関数を実行 (最後の`()`が必要なことに注意)
* __関数リテラルはクロージャなので、関数リテラル内から外側の関数内で定義した変数を参照可能__


### メソッド
* __structだけでなく、既存の型にもメソッド定義可能__ `func (i *int) hoge() int` みたいにintにメソッドを追加するとか
    * __メソッドはポインタとインタフェース以外すべての型に定義することができる__
    * 関数に対してメソッドを書くことも可能


### defer
* 遅延関数呼び出し - __関数がリターンする直前に、その中でdefer指定した関数の呼び出しが行われるようにスケジューリング__
    * リソースのclose処理とかで有用
    * finallyに近いイメージ
* defer指定された関数の引数(関数がメソッドであれば、レシーバも含む)は、関数の実行時ではなく、deferを実行したときに評価される
* defer指定された関数が複数ある場合、LIFO順で実行される


### 型
* 型アサーション - 変数vが型Tであるかをアサーションする場合、カンマOK慣用句を使って`vi, ok := v.(T)` と書く
  * 成功時はviにはvが指定した型に変換されたインスタンスが、okにtrueが設定される
* ある型の値/オブジェクトを特定の型に __変換__ したいとき(Goでは __変換__ と呼んでいるが、要はキャスト)
    * `a := T(realTimeValue)`と書く - これはプリミティブ型と同じ書き方で、realTimeValue は例えばTの定義が`type T int`ならintの値を指定する
    * [Conversions - golang.jp](http://golang.jp/go_spec#Conversions)


#### String()メソッド, fmt.Stringerインタフェース
* Javaで言うtoString
* __`fmt.Stringer`インタフェースを実装したことになる__ = __fmr.Println 等の引数として使用できるようになる__


### ネーミング
* __名前(トップレベルの型名、関数名、メソッド名、定数名、変数名、構造体のフィールドおよびメソッド名)の先頭一文字が大文字になっていれば、パッケージの利用者側から参照可能)__
    * 外部パッケージから可視状態であることを __エクスポートされた(exported)__ 状態と呼ぶ
* 大文字にしなければ、それが定義されているパッケージ内からしか参照できない


### 非同期処理 チャネル, ゴルーチン
[素数(プロセスと平行通信プログラミング) - golang.jp](http://golang.jp/go_tutorial#index12)

[並列性 - golang.jp](http://golang.jp/effective_go#concurrency)

* __Go言語では、共有変数をチャネル上で引き回し、実行中の異なるスレッドからは実際に同時アクセスさせないという今までとは異なる手法を推奨している__
    * Goの並行処理は分散メモリモデル - "共有メモリを使った通信は行わず、代わりに通信を使うことでメモリを共有するようにしてください"
* ゴルーチン - Go言語は独自のプロセス/スレッド/軽量プロセス/コルーチンモデルを持っていて、表記上の混乱を避けるためGo言語の平行実行処理をゴルーチン(goroutines)と呼ぶ
    * [コルーチン - Wikipedia](http://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%AB%E3%83%BC%E3%83%81%E3%83%B3)
    * ゴルーチンのモデル/役割 = __同一アドレス空間内で他のゴルーチン同士を並列実行すること__
    * `go [関数]` - バックグラウンド/非同期処理=ゴルーチンを開始する, キーワード`go`を付けて関数を呼ぶ
        * __呼ぶ関数は定義済関数でも匿名関数(クロージャ)でもよい__ (後者の場合は最後に`()`を付けて呼び出すのを忘れないように)
        * Unixシェルにてバックグラウンドでコマンドを実行するために使用する「&」表記とほぼ同じ
            * __つまりシェルの`wait`に相当する機能がある？ => ある__
    * __無限ループ内でゴルーチンを起動__ はよく見るイディオム。サーバーを立てる場合とか
* チャネル - 2つの並列動作している処理をつなぐ __通信チャネル__ , CSPが基になっている, Unix shellとの双方向パイプ
    * 完了通知を行う際に用いる
    * `chan` - チャネルを表す型. `chan int`型は "int型の値をやり取りするチャネル型" を表す
        * 新しいチャネル(変数)の割り当てには`make`を使う (makeの節を参照),
            * `ch1 := make(chan int) // バッファなしチャネル`
            * `ch2 := make(chan int, 0) //バッファなしチャネル`
            * `ch3 := make(chan *os.File, 100) // バッファ有りチャネル`
    * `<-` - チャネルとの値送受信を行う演算子。 `ch <- i`(送信), `j := <-ch`(受信(と代入))
        * 受信側は、そのチャネルへの送信があるまで(受信可能なデータが来るまで)そこでブロック(待機)される
        * 受信の際にカンマOK慣用句を使って`j, ok := <-ch`と2つの値を代入した場合、 __`ok`には「チャネルがcloseされていればfalse、それ以外ではtrue」がセットされる__
            * `database/sql/sql.go` - DB接続を行う関数にチャネルを渡し、その結果判定で`ret, ok := <-ch`という受け取り方をしている
        * 送信側は、、、
            * __バッファリングされていないチャネルへの送信は、受信側の準備ができたときに実行される__ , 逆に言うと受信側が準備されるまでブロックされる
            * バッファリングされているチャネルへの送信は、バッファに空きがあるときに実行される, 逆に言うと受信側が準備出来てなくても送信が動く
                * バッファに空きがある＆受信側の処理が無い -> __この状態でも送信だけ動く__ ということ
                * バッファに空きがない状態で送信を行った場合、 __空きが出来るまでブロックされる__
            * __スループットを制限するようなケースで、このバッファ有りチャネルの仕様をセマフォのように使うことが出来る__
                * ゴルーチンの呼び出し数を制御しても同様の機能を実現可能
            * __同じゴルーチン内(非同期なゴルーチン呼び出しが1つも無い関数呼び出しチェーン)で、送信 => 受信を逐次に書くと、デッドロックが発生する__
                * チャネルに値を送信すると、 __送信を行ったゴルーチンが待ち状態になる__
                * プログラムの実行状態に唯一存在するゴルーチンが停止したため、処理を進行させるゴルーチンが無くなったことによりデッドロックと判断される

                ```
                # ゴルーチン呼び出しが1つもなく、mainからの呼び出しチェーンが1つのゴルーチンで実行されている

                goroutine 16 [chan send]:
                main.sub2(0x20817e0c0)
                        /Users/hoge/gopath/src/github.com/huga/golang_first/main_para.go:150 +0x184
                main.sub1()
                        /Users/hoge/gopath/src/github.com/huga/golang_first/main_para.go:138 +0x42
                main.main()
                        /Users/hoge/gopath/src/github.com/huga/golang_first/main_para.go:17 +0x1a
                ```

            * [送信ステートメント - golang.jp](http://golang.jp/go_spec#Send_statements)
    * `close(ch)` でチャネルをクローズする, クローズ = __今後値が送信されないよう印をする__
        * ch が受信専用チャネルの場合にクローズしようとするとエラーになる
        * ch がクローズ済でその後送信や再クローズしようとするとランタイムパニックを引き起こす
        * close を呼び出し、事前に送信された値をすべて受信し終えた後で受信操作を行うと、そのチャネル型のゼロ値が __ブロックされることなしに__ 返される
            * __未受信の値をチャネルに残っている状態でclose を呼び出した場合、その後も受信は可能__
    * __チャネルは値(first-class value) である__ = メモリを割り当て可能で、他の型の値と同じように引き回すことが出来る
    * 関数の引数としてチャネルを使用する場合、 __受信専用/送信専用のチャネルを定義可能__
        * `func a(ch1 <-chan int)` - ch1からの受信のみ可、`func b(ch2 chan<- int)` - ch2への送信のみ可
    * ！！ ___チャネルを渡してゴルーチンを起動・起動元の後続処理でチャネルの受信(送信)処理・起動先ゴルーチンでチャネルへ値を送信(受信)___ とりあえずこれが基本形！！
        * 起動元で受信処理書いておかないとデッドロックするよ
        * 別ゴルーチンを起動する形にしないとデッドロックするよ
    * ___チャネル内の未受信要素数 って取得できるんだっけ？___ (無かったらWaitGroupのAdd/Done辺りを使う方法で代替するしか無いか)
* `select`ステートメント - `case`でリストされた複数の通信(チャネル)のうち処理が続行可能な方を選択する
    * すべてがブロックされていれば、どれかが続行可能になるまで待機する
    * どちらも続行可能なら __どちらかがランダムに選択される__
    * 送信の場合の挙動に注意
        * __チャネルと値の式は、ともに通信を開始する前に評価される__ ので、送信が実行可能となるまで通信はブロックされる

    ```go
    select {
    case req := <-service:
      go run(op, req)
    case <-quit:
      return
    case chX <- valX:  // 送信 : valXが評価されてから通信を開始（しようとする） -> select文実行のタイミングではまだ通信出来る状態ではないという事 -> 結果としてdefault句があるとそちらが動く
      return
    default:  // switchと同様に他のどのケースも実行できないないときに実行される=default節を定義しておけばselectがブロックすることは無くなる
    }
    ```

#### 複数CPUを使った並列化
* 環境変数`GOMAXPROCS`に使用するコア数を設定して実行すると実現可能
    * 一般的には`runtime`パッケージをインポートし、`runtime.GOMAXPROCS(runtime.NumCPU())`する


#### 並行処理実装時に利用する主なパッケージ・型
* `sync`パッケージ
    * `WaitGroup`型 - 起動したゴルーチンの完了を待ちたい場合に使用する
        * `Add()` - WaitGroupへの追加, 内部的には __カウンターを1加算__ している
        * `Done()` -  WaitGroupを1つ完了, 内部的には __カウンターを1減算__ している
        * `Wait()` - 全WaitGroupが完了するまで（カウンターが0になるまで）処理を待機する
            * シェルスクリプトの`&`と`wait`による制御と同じ
    * `Once`型 - 複数起動したゴルーチンで1回だけ実行したい処理を定義する際に使う
        * `Do()` - 1回だけ実行したい __関数__ をセットする
    * `Mutex`型 - 
        * `Lock()` - 
        * `Unlock()` - 
    * `RWMutex`型 - 
        * `Lock()` - 
        * `Unlock()` - 
        * `RLock()` - 
        * `RUnlock()` - 
        * `RLocker()` - 
    * `Cond`型 - 
        * `Broadcast()` - 
        * `Signal()` - 
        * `Wait()` - 
    * `Pool`型 - 
        * `Pool()` - 
        * `Put()` - 
* `time`パッケージ
    * `After()`
    * `AfterFunc()`
    * `Sleep()`
    * `NewTimer()`
    * `Time`型 - 
        * `Second()`
    * `Timer`型 - 
        * `C`変数 - __公式のパッケージドキュメントには記載が無いが__ 、 "現在時刻を送信するチャネル"
* `runtime`パッケージ
    * `NumCPU()`
    * `NumGoroutine()`
    * `LockOSThread()`

#### (参考ソース)
* `bufio/bufio_test.go`
    * `select { case ... }`文で`case <-time.After(time.Second) ...`というcase節を書くと、 __ゴルーチンのタイムアウトエラーのハンドリング__ が可能
* `database/sql/sql_test.go`
    * `defer runtime.GOMAXPROCS(runtime.GOMAXPROCS(maxProcs))` - 関数return時にGOMAXPROCSを(確実に)元に戻す為のtips
    * L1749で`defer close(reqs) // reqsはboolのチャネル`と、close関数をチャネルを引数にして実行している
    * L1753で`for _ = range reqs // reqsは同じく`と、`range`の対象にチャネルを使っている
* `go/printer/printer_test.go`
    * L163で "10秒経ったらチャネルに送信" するだけのゴルーチンがあって、後続のselect文でこのチャネルからの受信をエラー(タイムアウト)判定に使ってる
* `testing/benchmark.go`
    * `go func() {}`して起動した関数の最初に`defer wg.Done()`しておくと、WaitGroupのカウンター減算漏れを確実に防ぐことが出来る


### エラー処理
* `panic` - 実質的にプログラムを停止させるランタイムエラーを作成
    * panicが呼び出されるとカレントの関数を停止し、ゴルーチンのスタックの巻き戻しを開始し、その途中、遅延指定されている関数をすべて実行する
    * プログラム停止時に出力するため、任意の型(たいていは文字列)の引数をひとつ受け取る
    * 例えば無限ループから抜けるなどの、どうしようもできない事態が起きたことを示す手段でもある
    * __問題点を隠せるか回避できるのであれば、プログラム全体をダウンさせるより、実行を継続させるべき__
        * ただし、初期化においてはそうとは限らず、ライブラリが自身のセットアップがどうしても行えないときは、panicを起こすことは合理的
    * 配列のインデックスが範囲外であるときや、型アサーションに失敗したような場合にも暗黙的にpanicが呼ばれる
* `recover` - `panic`により巻き戻しが始まっているゴルーチンの制御を取り戻し、通常の実行を再開させることが可能になる
    * __recoverを呼び出すと、巻き戻しを停止し、panicに渡された引数が返る__
    * __巻き戻り中に実行できるコードは、defferで遅延指定された関数内のみ__ なので、recoverは遅延指定された関数内でのみ役立つ
        * __遅延指定された関数から呼び出されない限り、recoverは常にnilを返す__

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```


### リフレクション

#### reflectパッケージ
* `Type`型 - Go言語の型のランタイム表現。すべての型がここでリストされるメソッドを実装している。内部的にはinterface型である型で、`reflect.TypeOf(i)`で取得
    * 構造体のタグもこの`Type`を経由して取得する事が出来る
    * `Method(i int)`メソッド - 
    * `Name()`メソッド - この型の名前を返す
    * `PkgPath()`メソッド - パッケージの完全なインポートパスを返す
    * `Size()`メソッド - この型の値を格納するのに必要なバイト数を返す
    * `String()`メソッド - 型の文字列表現を返す。文字列表現では、短縮したパッケージ名が使われるので、型の名前がユニークになる保証は無い
    * `Kind()`メソッド - この型のKind(種類)を返す
    * `Elem()`メソッド - この型の要素のTypeを取得する
        * __`Kind`が 配列、チャネル、マップ、ポインタ、スライス 以外のTypeで実行するとpanicを引き起こす__
    * `Field(int)`メソッド - この型が構造体の場合、i番目のフィールドの情報を`StructField`型で返す。 __`Kind`がポインタのTypeで実行するとpanic__
        * `FieldByIndex([]int)`メソッド - 
        * `FieldByName([]int)`メソッド - 
* `Value`型 -  ザックリ言うと値を表す型。内部的にはstruct型である型で、`reflect.ValueOf(i)`で取得
    * `Type`インタフェースのメソッドを実装している
* `Kind`型 - `Type`型が表現している型の種別を表す型
    * 例えばfloat32のTypeは FloatType だが、Kindは Float32 になる
* `Method`型 - 
* `StructField`型 - 構造体のフィールドを表すstruct型
    * __フィールドのタグはこの型の`Tag`フィールドにセットされる__
* `StructTag`型 - 構造体のフィールドのタグを表すstring
    * __実際にタグを記述する際、書式に則って記述すると、後述の`Get(key string)`メソッドでkey-valueを意識したアクセスが可能になる__

    ```go
    type S struct {
      // スペース区切りで key:"value" 形式のペアを羅列する
      // タグ全体をバッククォート ` で囲む
      // (タグがこの書式に則ってない場合に Get(key) を実行すると、空文字が返る)
      F string `species:"gopher" color:"blue"` // Get("species") で "gopher" を、Get("color") で "blue" を取得可能
    }
    ```

    * `Get(key string)`メソッド - `key`に該当するタグの値(=string)を取得する


#### unsafeパッケージ


### ネットワーク、通信


#### netパッケージ



### 暗号化関連

#### cryptoパッケージ



#### encodingパッケージ


### ヒープとスタック


### ソースコード解析関連

#### goパッケージ


## goコードの書き方
[Goコードの書き方 - golang.jp](http://golang.jp/code)

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


## 公開されているツール/ライブラリを使う
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

#### 

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




## goプロジェクトを公開する
githubに公開することを例に

* 参考 : [drone/drone](https://github.com/drone/drone)
    * README
        * Travis CIやdroneのCIのbadge(=ステータスアイコン)を表示する (これはgolangという枠に限った話ではないが)
        * godoc のReferenceのbadge(=リンクアイコン)を表示する
            * godoc.orgの tools ページにHTMLとmarkdownで使えるsnippetが公開される
            * 例 : [ipcl tools - GoDoc](http://godoc.org/github.com/jpshadowapps/ipcl?tools)
* [TODO] よくあるディレクトリ構成
* [TODO] 開発者向けの情報共有
    * バイナリとして使用する際の手順
    * ライブラリとして使用する際の手順
    * pull req送りたい時のローカルプロジェクトセットアップ手順
    * ビルド手順の提供（Makefile ?)
* [TODO] 利用者向けの情報共有
    * バイナリとして使用する際の手順



## package documentの歩き方
[Packages - The Go Programming Language](http://golang.org/pkg/)

```
Package net

import "net"  // このパッケージを使用したい場合のimport文
Overview  // 概要とちょっとしたサンプルコード
Index  // 目次(後述)
Examples  // サンプルコード
Subdirectories  // サブパッケージ一覧
```

### Index
パッケージドキュメントの目次

```
Constants  // 定数一覧
Variables  // 変数一覧
func Interfaces() ([]Interface, error)  // トップレベルの関数(パッケージ関数？)
type Interface  // 型の定義
    func InterfaceByIndex(index int) (*Interface, error)  // 関数 (レシーバがない)
    func InterfaceByName(name string) (*Interface, error)
    func (ifi *Interface) Addrs() ([]Addr, error)  // メソッド (レシーバが明示されている)
    func (ifi *Interface) MulticastAddrs() ([]Addr, error)
```

#### Constants
パッケージ配下で定義されている定数群

* ソース

```
const (
        IPv4len = 4  // 型省略なケース = 式によって型が決まる
        IPv6len = 16
)

IP address lengths (bytes).
```

* (書式)

```
ConstDecl      = "const" ( ConstSpec | "(" { ConstSpec ";" } ")" ) .
ConstSpec      = IdentifierList [ [ Type ] "=" ExpressionList ] .

IdentifierList = identifier { "," identifier } .
ExpressionList = Expression { "," Expression } .
```

#### Variables
パッケージ配下で定義されている変数群

* ソース

```
var (
        IPv4bcast     = IPv4(255, 255, 255, 255) // broadcast
        IPv4allsys    = IPv4(224, 0, 0, 1)       // all systems
        IPv4allrouter = IPv4(224, 0, 0, 2)       // all routers
        IPv4zero      = IPv4(0, 0, 0, 0)         // all zeros
)

// Well-known IPv4 addresses


var (
        IPv6zero                   = IP{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0}
        IPv6unspecified            = IP{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0}
        IPv6loopback               = IP{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1}
        IPv6interfacelocalallnodes = IP{0xff, 0x01, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0x01}
        IPv6linklocalallnodes      = IP{0xff, 0x02, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0x01}
        IPv6linklocalallrouters    = IP{0xff, 0x02, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0x02}
)

// Well-known IPv6 addresses

var ErrWriteToConnected = errors.New("use of WriteTo with pre-connected UDP")
```

* (書式)

```
VarDecl     = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
VarSpec     = IdentifierList ( Type [ "=" ExpressionList ] | "=" ExpressionList ) .
```
```
ShortVarDecl = IdentifierList ":=" ExpressionList .   // 省略形式
```

#### 型(type)
パッケージ配下で定義されている型群

* ソース

```
type Interface struct {
        Index        int          // positive integer that starts at one, zero is never used
        MTU          int          // maximum transmission unit
        Name         string       // e.g., "en0", "lo0", "eth0.100"
        HardwareAddr HardwareAddr // IEEE MAC-48, EUI-48 and EUI-64 form
        Flags        Flags        // e.g., FlagUp, FlagLoopback, FlagMulticast
}
```

* (書式)

```
TypeDecl     = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
TypeSpec     = identifier Type .
```
```
Type      = TypeName | TypeLit | "(" Type ")" .
TypeName  = identifier | QualifiedIdent .
TypeLit   = ArrayType | StructType | PointerType | FunctionType | InterfaceType |
      SliceType | MapType | ChannelType .
```

#### (トップレベルの)関数(func)
パッケージ配下で定義されている関数群

* ソース

```
func Interfaces() ([]Interface, error)
```

* (書式)

```
FunctionDecl = "func" FunctionName Signature [ Body ] .
FunctionName = identifier .
Body         = Block .
```
```
FunctionType   = "func" Signature .
Signature      = Parameters [ Result ] .
Result         = Parameters | Type .
Parameters     = "(" [ ParameterList [ "," ] ] ")" .
ParameterList  = ParameterDecl { "," ParameterDecl } .
ParameterDecl  = [ IdentifierList ] [ "..." ] Type .
```

#### メソッド(func)
type別のメソッド群

* ソース

```
func (ifi *Interface) Addrs() ([]Addr, error)
```

* (書式)

```
MethodDecl   = "func" Receiver MethodName Signature [ Body ] .
Receiver     = "(" [ identifier ] [ "*" ] BaseTypeName ")" .
BaseTypeName = identifier .
```

## go自体のソースをローカルで読みたい

* clone先が`$GOPATH`下である必要はないような気もするが

```
% cd $GOPATH/src/code.google.com/p
$ hg clone https://code.google.com/p/go
```



## gocon 

### Brad Fitzpatrik
* 90% perfect , 100% of the time
* fast for human : 動的スクリプト言語 > GO > Java, C etc
* fast for performance : 動的スクリプト言語 < GO ≒java, C
* Web frmaeworks - net/http is standard
* シェルスクリプトを置き換えるという用途
  * 小さいperlとか
  * os/exec
* ARMで動く
* __Camlistore__
* github.com/nf/sigoumey
* webapps, scripts, system admin, image processing. l/b servers, resbery PI, etc
* no shared librarilies1
* GC - heap in 1.2, most stacks in 1.3, more in 1.4
* tools/godep


### Rui Ueyama
社内システムをGoで書き換えた話, App EngineアプリのPythonからGoへの移行

* 静的な型付けによる安心感

### @davecheney
bit.ly/1wwG7AT


* なぜGo？ concurrency, ease of deployment, performance
    * データ構造がコンパクト
* デッドコードの最適化

```go
func Test() { return false }

funct Main() {
  if Test() {
      // unreachable! - コンパイラで Test() 呼び出しの部分は "if false" に最適化される
  }
}
```

* ループ内での関数呼び出しで、呼び出しのたびに呼び出しstackがcreate(とdelete)される問題があった
    * ver1.3からは1回目の呼び出し時に呼び出し元/先を含めたスタックのまとまりがコピーされ、create/deleteが繰り返されるという挙動ではなくなる
    * パフォーマンスが改善する

```go
func G(items []Item) {
  for item, _ := range Items {
    H(item)  // ここでstackのcreate/deleteが呼び出しのたびに発生する
  }
}

func H(item Item) {
  // hogehoge
}
```

### tanaka-san @ hatena
Mackerel.io でGoを使った話

* mackerel-agentをGoで書いてる
* サーバの情報を収集, REST APIでpostする　というお仕事を粛々とやるサービス
* クロスコンパイラ, 依存性小, 1 binary, small footprint, セットアップ容易
* `ticker`で1分ごとに起動
* /proc/stat の中を開いてパースしたりを頑張って作ってる
    * 文字列操作はperlやrubyが懐かしくなることも...
* コレクタの並列化
    * goroutineやchannelを駆使して簡単に書けるのが良い
* __forループの処理でよく見かける `range` __
* rpmパッケージのbuildでdockerを使ってる
* daemonizeは nohup と & を駆使してやってる（愚直なやり方）
* Pros/Cons
  * Pros - channelとgoroutineがGREAT, 色々なアーキテクチャのサポートが容易, 
  * Cons - 文字列操作処理をガリガリやろうとすると大変かも, JSONのパースも大変

### rosylilly-san

### matsumoto-san @ gunosy
n百万人をGoで捌いてみた

### plan9user-san
Go駆動開発で超速Pushエンジンを作った話


## TODO
pprof
memstats
pkg/expvar パッケージ
