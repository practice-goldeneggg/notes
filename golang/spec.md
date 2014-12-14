## 主な言語仕様
とりあえず下記を読んどくべし

* [Goプログラミング言語仕様 - golang.jp](http://golang.jp/go_spec)
* [実践Go言語 - golang.jp](http://golang.jp/effective_go)


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


### 文字列
* `string`型
* 文字列型の要素はbyte型データを持っているため、一般的なインデックスを使ったアクセスが可能
* `len(strvar)`関数で文字列長を調べることが出来る
* 文字列に対して`range`のループを記述すると、 __UTF-8エンコードでUnicodeの各文字を取り出す__ `for pos, char:= range "日本語" { ... }`
* リテラル
    * 未加工文字列リテラル``hoge`` - バッククォートで囲んだ文字列リテラル, バックスラッシュや改行もそのまま含まれる
    * 解釈有文字列リテラル`"hoge"` - ダブルクォートで囲んだ文字列リテラル, バックスラッシュはエスケープ用、や改行を含めることは出来ない


### 配列
* goの配列は __値__ である
    * cのように "先頭の配列要素へのポインタ" というわけではない
* 配列変数
    * `var hoge [3]int`(サイズが3)
    * `var hoge [3]int{1,2,3}`(サイズが3で要素が固定)
* 配列リテラル
    * `[2]int{1,2}`(要素数＋要素)
    * `[...]int{1, 2}`(要素数を数えず ... で省略)  __何も書かない場合との違いは？ => 何も書かないとスライス__


### スライス
[SliceTricks - go-wiki - How to do vector-esque things with slices. - Go Language Community Wiki - Google Project Hosting](https://code.google.com/p/go-wiki/wiki/SliceTricks)

* スライスは配列と似ているが違う。 __サイズ情報を持たず、配列の要素を "部分的に" "参照している" のがスライス__
    * サイズを明示的に持たず([]と[10]の違い)
    * 作成元である通常の配列(匿名であることが多い)を部分的に参照している
    * 配列だけではデータを共有はできないが、 __複数のスライスが同一の配列を参照することでデータ共有が可能になる__
* __どのような配列・サイズであっても要素型が同じであれば、それを参照するスライス変数を宣言できる__
    * __スライスは実体への参照である__
    * つまり __`[3]int{1, 2, 3}`は配列で`&[3]{1, 2, 3}`はスライス__
* 通常の配列より柔軟で、実体への参照であり、かつ効果的であるためGo言語のプログラム内で多用される
* 関数に配列を受け渡すときは、大抵はパラメータをスライスとして宣言する
    * "関数を呼び出すとき、効率のため配列をスライスし、スライス参照を作成し、それを渡すようにしてください"
* スライス変数 - `var hoge []int`
* スライスリテラル - `[]int{1, 2}`(要素のみ)
    * スライスを引数に取る関数を呼ぶ際はこの書き方で`funcname([]int{1, 2})`と呼ぶのが慣例
* スライス式は `a[low : heigh]`形式

#### append関数
* スライスの末尾に要素を追加したスライスを返す関数
* __スライスにスライスを追加する場合は`append(sliceA, sliceB...)`と`...`を付ける。__ `...`を記述しないと、型が一致しないためコンパイルできない


### マップ
* コレクションライブラリではなく組み込みの機能
* `map[KEY_TYPE]VALUE_TYPE`形式で定義
* mapの初期化
    * `map[string]int{"one":1, "two":2}` - `{KEY:VALUE}`形式で初期化子を書く
    * `make(map[string]int)`, `make(map[string]int, 10)` - makeで空の新しいマップを作る, 第2引数はキャパシティの初期値
        * マップ(nilマップを除く)は格納する要素に合わせてサイズを拡張する
* mapの要素数は`len(m)`で確認可能
* mapの要素(key-valueの組み合わせ)の削除は`delete(m, key)`で可能
* __カンマOK慣用句 を使うと、mapの値と存在(true/false)を取得可能__ 存在しない場合は値にはゼロ値がセットされる
    * `v, ok = hogeMap[key] // keyが存在すればokはtrue`
    * 値をブランク識別子(`_`)にすると、値を破棄できる。存在チェックだけしたい場合に使える
* mapから __値を__ 削除する場合は、 カンマOK慣用句の右辺左辺を入れ替えればよい `hogeMap[key] = 0, false`
* mapに対して`range`のループを記述すると、 __key, valueを個々に取り出せる__ `for k, v := range kv { ... }`


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


### golangはオブジェクト指向言語ではない(構造体,インタフェース,埋め込み & 応用テク)

#### 構造体
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
        * __構造体のフィールド変数はpublicな・exportされた・先頭が大文字の・でなければ機能しない__
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

##### コンストラクタと複合リテラル
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

#### インタフェース
* __interfaceで定義したメソッドを実装している型であれば（たとえ他のメソッドを有していても）そのinterfaceを実装していることになる__
    * 例えば`type IHoge interface { huga() }`というinterfaceがある場合、`type Hoge struct {...}`のメソッドとして`huga()`を実装すれば、型Hogeはinterface IHoge を実装していることになる
* オブジェクト指向系言語のinterfaceとは勝手が全く違う
    * ラップ, コンポジションがベース
* Reader/Writer あたりのソースが参考になる
* インタフェースに名前をつける際、メソッド名にer(またはr)をつけるというのがGo言語の習わし
* インタフェース型の変数を`interface{}`と波カッコ付きで定義してるのをよく見るが、これは __波カッコ内のメソッド定義を省略している="任意のメソッドを実装している型"__ という意味
    * 逆に言うと`var hoge interface{ Aho() }`という型定義の場合は "Aho()というメソッドを実装している任意の型" を定義したことになる


#### 埋込み
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

#### 既存型を元にした型宣言
* `type <識別子> <基礎型>` の構文, 基礎型が既存の型
* 例としてはこういうの => `type NewMutex Mutex`
* __新たな型を定義してメソッドを追加__ したりといった既存の型の拡張テクニックとして使われる
* 新たな型と元となる型（基礎型）の間にはルールがある
    * 基礎型がstructの場合、そのフィールドはそのまま継承される
    * 基礎型から __メソッドは継承しない__
    * `var baseVar 基礎型`のように基礎型として宣言した変数に新たな型は代入 __できない__
        * ただし __基礎型にキャストすれば代入可能__, `baseVar = 基礎型(newVar)`といった具合
    * インタフェース型およびコンポジット型の要素が持つメソッド群は、そのまま変更されることなく残る (???)

    ```go
    // Mutexは、LockとUnlockメソッドを持つデータ型
    type Mutex struct         { /* Mutex fields */ }
    func (m *Mutex) Lock()    { /* Lock implementation */ }
    func (m *Mutex) Unlock()  { /* Unlock implementation */ }

    // NewMutexはMutexと同じ構成ではあるがメソッド群は空
    type NewMutex Mutex

    // PtrMutexのベース型のメソッド群は、そのままだが、
    // PtrMutexのメソッド群は空。
    type PtrMutex *Mutex

    // *PrintableMutexのメソッド群には匿名フィールドMutexに
    // バインドされているLockとUnlockメソッドが含まれる
    type PrintableMutex struct {
      Mutex
    }
    ```

* __基礎型には関数も指定可能__ (関数型って呼び方で良い？)
    * 代表例が`http.HandlerFunc`で、その定義は`type HandlerFunc func(w http.ResponseWriter, r *http.Request)`となっている
        * __関数(型)にメソッドを定義する事も可能__ で、`HandlerFunc`は`ServeHTTP(w ResponseWriter, r *Request)`メソッドを定義している
* [インタフェースの実装パターン #golang - Qiita](http://qiita.com/tenntenn/items/eac962a49c56b2b15ee8)

#### (参考ソース)
* [profile/profile.go at master · davecheney/profile](https://github.com/davecheney/profile/blob/master/profile.go#L83)
    * 関数の戻り値を"指定したメソッドを実装している型(インタフェース)"にしたい場合
        * __特定のinterface型を指定するのではなく、無名(匿名?)interfaceで記述する__
        * `func hoge() interface { Huga() } { 関数本体部 }`という書き方で書く
            * この時関数本体部で返す戻り値が`Huga()`を実装している型で無ければならない




### 型
* 型アサーション - interface{}型からリアルの型への型変換/キャスト
    * __interface{}型の変数viが型Tであるかをアサーションする__ 場合、カンマOK慣用句を使って`v, ok := vi.(T)` と書く
    * 成功時はvにはviが指定した型に変換されたインスタンスが、okにtrueが設定される
* ある型の値/オブジェクトを特定の型に __変換__ したいとき(Goでは __変換__ と呼んでいるが、要はキャスト)
    * `a := T(realTimeValue)`と書く - これはプリミティブ型と同じ書き方で、realTimeValue は例えばTの定義が`type T int`ならintの値を指定する
    * [Conversions - golang.jp](http://golang.jp/go_spec#Conversions)
* 型switch - 変数の型によって条件分岐するswitch文
    * 変数vの型は`v.(type)`で取得できる, 予約後`type`を型名の代わりに使ってる
    * switch文を書く際に、省略形式の変数宣言`switch t := v.(type)`が書ける
        * スコープは`case`と`default`節内
    * caseに型を複数指定した場合は、変数は型スイッチガードの式が表す型になる
    * caseにはnilも指定可能

```go
switch i := x.(type) {
case nil:
  printString("x is nil")
case int:
  printInt(i)  // iはint
case float64:
  printFloat64(i)  // iはfloat64
case func(int) float64:
  printFunction(i)  // iは関数
case bool, string:
  printString("type is bool or string")  // iはinterface{}
default:
  printString("don't know the type")
}
```


#### String()メソッド, fmt.Stringerインタフェース
* Javaで言うtoString
* __`fmt.Stringer`インタフェースを実装したことになる__ = __fmr.Println 等の引数として使用できるようになる__


### forループ
* __Goにはwhileは無い。ループはforのみ__
* 複数の変数を回したい時は、CやJavaのように`,`で区切って個別に記述は出来ない。カンマ演算子が無い為
    * 複数の変数を回したい時はこう書く。 複数変数定義時のカンマの使い方と同じで、 __一度に代入するイメージ__ `for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1`


### range
* 要素数のループを繰り返す, 文字列,配列,スライス,map,channel で利用可能
* `for i := 0; i < len(a); i++ { ... }` は `for i, v := range a { ... }`と書ける, __iにはループのインデックスが, vには繰り返し要素が順に代入される__
    * インデックス(key)を使用しない場合はiを省略する, あるいは`for _, v := range a {...}`とプレースホルダを使用して書ける。が、keyを使用しないということを明示する意図で`_`を書くのがgoの慣例？
* 文字列のrangeは __position(key)と文字(value)__ を取り出せる
    * 逆に言うと、 __文字列のrangeで`for s := range strs {...}`って具合にパラメータを1つのみ書いた場合、（意図としては文字列配列の要素が欲しいんだろけど実際には）ポジション番号しか取得できない__
        * これを回避する為には`for _, s := range strs {...}`と書く。ポジション番号は不要なので捨てる


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
    * 関数内では`[]T`型（型Tのスライス）として扱われる
    * 可変長変数リストを持つ関数をネスト呼び出しする際は、呼び出す関数に`(v...)`という形式で引数を渡す。こうしないと、vは、単にひとつの(新しい)スライスとして扱われる

```go
// Greeting関数内のwhoの値は[]string{"Joe", "Anna", "Eileen"}になります。
func Greeting(prefix string, who ...string)
Greeting("hello:", "Joe", "Anna", "Eileen")

// スライスsを使って呼び出し
// 後ろに...を伴えば、...Tパラメータの値は変更されることなく渡される
s := []string{"James", "Jasmine"}
Greeting("goodbye:", s...)
```


#### init関数
* セットアップのためにパラメータを持たない関数initを定義することができる
    * 初期化中にゴルーチンを起動することはできますが、初期化が完了するまで実行は開始しない。すなわち、初期化処理中に実行されるスレッドは常にひとつだけ
* __インポートしているパッケージの初期化、変数の初期化 が評価された後に呼び出される__


#### 関数リテラル
[関数リテラル - golang.jp](http://golang.jp/go_spec#Function_literals)

* `f := func(x, y int) int { return x + y }` - 変数に代入
* `func(ch chan int) { ch <- ACK }(replyChan)``func() { os.Exit(1) }()` リテラルの関数を実行 (最後の`()`が必要なことに注意)
* __関数リテラルはクロージャなので、関数リテラル内から外側の関数内で定義した変数を参照可能__


#### 実装テク
* ローンパターン, decorator（ぽいこと）
    * [共通作業をまとめちゃおう (withContext wrapper functions)](http://qiita.com/ksato9700/items/6228d4eb6d5b282f82f6#2-10)


### メソッド
* __structだけでなく、既存の型にもメソッド定義可能__ `func (i *int) hoge() int` みたいにintにメソッドを追加するとか
    * __メソッドはポインタとインタフェース以外すべての型に定義することができる__
    * 関数に対してメソッドを書くことも可能
* レシーバを値にすべきかポインタにすべきか？
    * メソッド実行後にレシーバの状態を変えたければポインタ一択
    * [ポインタ vs. 値 - golang.jp](http://golang.jp/tag/effective-go/page/2)


### defer
* 遅延関数呼び出し - __関数がリターンする直前に、その中でdefer指定した関数の呼び出しが行われるようにスケジューリング__
    * リソースのclose処理とかで有用
    * finallyに近いイメージ
* defer指定された関数の引数(関数がメソッドであれば、レシーバも含む)は、関数の実行時ではなく、deferを実行したときに評価される
* defer指定された関数が複数ある場合、LIFO順で実行される


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

#### 複数コア(CPU)を使った並列化
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

### 書式フォーマット(printf系で使用可能なもの)
[Printing - fmt - The Go Programming Language](https://golang.org/pkg/fmt/#hdr-Printing)


### 関数・変数のネーミング
* __名前(トップレベルの型名、関数名、メソッド名、定数名、変数名、構造体のフィールドおよびメソッド名)の先頭一文字が大文字になっていれば、パッケージの利用者側から参照可能)__
    * 外部パッケージから可視状態であることを __エクスポートされた(exported)__ 状態と呼ぶ
* 大文字にしなければ、それが定義されているパッケージ内からしか参照できない


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

### ヒープとスタック
