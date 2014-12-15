## ビルトイン関数
[builtin - The Go Programming Language](http://golang.org/pkg/builtin/), 詳細は各サブ節で書くとして、どんなビルトイン関数があるかは左記URLを参照


## 主なパッケージと提供機能

### 入出力
* Reader/Writerを __パイプ__ で接続する `io.Pipe() (*PipeReader, *PipeWriter)`
    * メモリ内に同期パイプを作成し、io.Readerを必要としているコードとio.Writerを必要としているコード間を接続する
    * __片方の読み込みは、もう一方の書き込みに対応し、データのコピーは二者間でバッファリングされることなくダイレクトに行われる__


### 外部コマンド/OSコマンドの実行 (& 標準入出力の取り扱い)


#### 利用する主なパッケージ
* `os/exec`
    * __子プロセスを起ち上げてコマンドを実行する__
    * `Command(name string, arg ...string) *Cmd`関数 - コマンド名と使用する引数を指定して`Cmd`を取得する関数
        * __コマンドのオプションは第2引数のargで指定する__
    * `Cmd`型 - 実行準備中もしくは実行中の外部コマンドを表す型
        * `Start() error` - コマンド実行。 __完了を待たない__
        * `Wait() error` - `Start`で実行したコマンドの完了を待つ。
        * `Run() error` - コマンド実行。 __完了するまで待つ。__  __実行されて、I/Oも正常で、終了コードが0なら、戻り値はnil__ になる。そうでなければ`*ExitError`型か他のerrorが返る
        * `Output() ([]byte, error)` - コマンド実行＋標準出力に出力された結果の取得
            * __一度任意のCmdで`Output`を実行すると、そのプロセスが完了するまで標準出力を握りっぱなしにするっぽい = もう1度Outputを呼ぶとエラーになる__
        * `CombinedOutput() ([]byte, error)` - コマンド実行＋標準出力/標準エラー出力に出力された結果の取得
        * `StdoutPipe() (io.ReadCloser, error)` - 実行したコマンドが標準出力に出力した結果をパイプして読み込む為の`Reader`(具体的には`ReadCloser`)を取得する
        * `StderrPipe() (io.ReadCloser, error)` - 実行したコマンドが標準エラー出力に出力した結果を (同上)
        * `StdinPipe() (io.WriteCloser, error)` - 実行するコマンドで必要な入力=標準入力の内容を編集する為の`Writer`(具体的には`WriteCloser`)を取得する
* `syscall`
    * `Exec(argv0 string, argv []string, envv []string) (err error)` - プロセスを起ち上げてコマンドを実行する
        * いわゆる `exec()` の動きをとる
        * `os/exec`と違って __起動元のGoプログラム・プロセスを "置き換える" 別プロセスを起ち上げて実行する__
            * コマンドプロンプト上から実行したのと同様の挙動になる（プロセス起ち上げ、結果出力）, 例えば`ls -l`とか
    * `ForkExec(argv0 string, argv []string, attr *ProcAttr) (pid int, err error)` - プロセスをforkしてコマンドを実行する
        * いわゆる `fork()` `exec()` の動きをとる


#### (参考ソース)
* [StdoutPipe - The Go Programming Language](https://golang.org/pkg/os/exec/#Cmd.StdoutPipe) のExample
    * `Command()`でCmdを構築 -> `StdoutPipe()`で結果の標準出力をパイプ -> `Start()`で実行 -> パイプから標準出力を読み込んでゴニョる -> `Wait()`で完了を待つ、 という例
        * ゴニョる, は具体的には標準出力への出力結果がJSONとして妥当かvalidationしている
        * 読み込みの際は`bufio.NewReader(reader)`でバッファリングされたReaderを構築して読み込む例が多い（この方が効率良さそうだし）
* `os/exec/exec_test.go`
    * `TestPipes`関数 - 標準入出力をパイプする処理の例
    * `TestCatStdin`関数 - 標準入力の内容をcatする処理の例


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
* `Listen(net, laddr)`関数 - 指定したネットワークをlistenする
    * `net` - stream-orientedなネットワークを指定可, tcp, tcp4, tcp6, unix, unixpacket, etc
    * `laddr` - ローカルアドレス, net = tcpなら`:PORT_NUM`形式でポート番号を指定するとか, net = unix ならsocketファイルのパスを指定
* `Listener`インタフェース - 



#### net/httpパッケージ

##### クライアントの実装


##### サーバーの実装
* `ListenAndServe(addr string, handler Handler)`関数 - 指定アドレス(ポート)でリクエストを待ち受けるサーバーを起動する, リクエストが来たらhandlerで指定したHandlerFuncを実行する
    * handlerは基本的にはリクエストパス毎に`Handle``HandleFunc`関数で登録するので、この関数の引数で指定することは稀
        * handlerがnilの場合、内部では`DefaultServeMux`型が使われている
* `Handle(pattern string, handler Handler)`関数 - patternで指定したパスへのアクセス時に実行するhandlerを登録する
* `HandleFunc(pattern string, handler func(ResponseWriter, *Requerst))`関数 - `Handler`と同様にhandlerを登録するが、指定するhandlerは関数のシグニチャが一致していれば型は何でも良い(=Handler型でなくてもよい)
* `Serve(l net.Listener, handler Handler)`関数 - 指定したListener(前述)へのアクセス時に実行するhandlerを登録する

* `Handler`インタフェース
    * `ServeHTTP(ResponseWriter, *Request)`メソッド - リクエストが来たら呼ばれる処理, 明示的に呼ぶ事も可能っちゃ可能
        * `http.HandlerFunc`型 (後述)
        * `http.ServeMux`型 (後述)
        * `http.Counter`型
        * `http.Chan`型
        * `http.cgi.Handler`型
        * `http.httputil.ReverseProxy`型
* `HandlerFunc`型 - `Handler`インタフェースを実装している __関数型__ ,`Handle`関数で指定するhandlerは（基本）この型で定義する, いわゆるHTTPハンドラ
    * `ServeHTTP(ResponseWriter, *Request)`メソッドを実装している, __functionがメソッドを実装している形で、メソッド内ではこのfunctionが実行されているだけ__
* `ServeMux`型 - HTTPリクエストのマルチプレクサ。登録されているパターンのリストと各リクエストのURLを比較し、URLと最も一致するパターンに登録されているハンドラを呼び出す
    * `Handle``HandlerFunc`の両関数は、内部で`DefaultServeMux`を使用している
* `Server`型 - `Handler`インタフェースを実装している

* チャネルの併用tips

#### net/http/httptestパッケージ
httpのテストをサポートするパッケージ



#### net/http/pprofパッケージ


#### net/http/fcgiパッケージ
FastCGIプロトコル実装パッケージ


#### syscallパッケージ
* `Socket(domain, typ, proto)`関数 - 



### 暗号化関連

#### cryptoパッケージ



#### encodingパッケージ


### ソースコード解析, ドキュメント作成

#### goパッケージ


##### go/docパッケージ
* go公式ドキュメント(golang.org)は、docパッケージの機能で生成されている


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
