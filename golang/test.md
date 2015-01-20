## テスト

### ブクマ
* [Test packages](http://golang.org/cmd/go/#hdr-Test_packages)
* [Description of testing functions](http://golang.org/cmd/go/#hdr-Description_of_testing_functions)
* [Description of testing flags](http://golang.org/cmd/go/#hdr-Description_of_testing_flags)

### 概要めも
* テスト実行は`go test`コマンド
    * 引数なしのデフォルト挙動は、カレントディレクトリのパッケージ内のテストをコンパイル・実行する
    * `-c` - `PKGNAME.test`という名前でテストファイルのコンパイル結果を出力する。実行はしない
        * `-o` - `PKGNAME.test`とは別の名前で出力する。実行もする
        * ___`-c`するときに`-bench`のようなフラグを指定するとバイナリの結果も変わる？___ => 変わらないっぽい
    * `-i` - テストが依存しているパッケージをインストールする。テスト実行はしない
    * `-exec`

    ```
    usage: go test [-c] [-i] [build and test flags] [packages] [flags for test binary]
    ```

* `**_test.go`という名前のgoファイルをテストコードとみなし、これらをコンパイルしてテストを実行する
* テスト結果はパッケージ単位で出力される（成否 + 所要時間）
* テストファイルを`*_test`というパッケージ名にすると、メインとは別パッケージとしてコンパイルされ、mainのテストバイナリと共にリンク・実行される
* テストパッケージはtemporaryディレクトリでビルドされるので、テストを行わないインストール作業には干渉しない
* オプション以外に`testflag`というテスト制御用フラグがある
    * __毎回`go test FLAG`するのは時間かかるしメンドイ。まずコンパイルしてテストバイナリを出力し、testflagを変えつつそのバイナリを実行する、という形にした方が良い__
        * testflag指定でテストバイナリを実行したい場合、フラグに`-test.`というprefixを付けて指定する
            * 例えば`-bench`なら`$ ./pkg.test -test.bench="Hoge"`とか`./pkg.test -test.bench="."`(全テスト) という形式で指定して実行する
    * __usageを見れば分かるが、`build flags`と`flags for test binary`がゴッチャにならないように__

### testing.T 型 (ユニットテスト)

* テスト実行時に指定可能なフラグ  (ベンチマークで使用可能なものもあり)
    * `-v` - 詳細出力。`t.Log`等でのログ出力が有効になる
    * `go help testflag`で確認できる


#### TestXXX テスト関数 (標準出力の内容をテスト)



#### ExampleXXX テスト関数 (標準出力の内容をテスト)


### testing.B 型 (ベンチマーク)

* ベンチマーク・テストを行う為の型
* `for i := 0; i < b.N; i++ {` というループを書いてその中で対象関数を呼ぶとかする
    * `b.N`はループ回数

```go
package main

import (
	"testing"

	"github.com/goldeneggg/benchwork/be"
)

var hmt = be.HMap{"key1": 1, "key2": 22, "key3": 333}

// ループをN回回して測定
func BenchmarkHoge(b *testing.B) {
	for i := 0; i < b.N; i++ {
		hmt.Keys()
	}
}

// パラレルに実行して測定
func BenchmarkHogeP(b *testing.B) {
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			hmt.Keys()
		}
	})
}
```

* `RunParallel`メソッド - 


#### ベンチマーク・テストの実行

* 起動
    * 通常のテストコマンドの末尾に`-bench <. | bench対象regexp>`を付加する(`.`は全てが対象)
        * `go test . -bench .` - ルートパッケージのテスト＆ベンチマークテストを行う
        * `go test ./... -bench .` - 全てのサブパッケージのテスト＆ベンチマークテストを行う
        * `go test ./... -bench HogeP` - 全てのサブパッケージのテスト＆テストメソッドに"HogeP"を含むベンチマークテストを行う
    * 結果の見方
        * `BenchmarkHoge  5000000    419 ns/op` - "テストメソッド名" "ループ回数" "ナノ秒/ループ"
    * その他のオプション
        * `-benchmem` - `go test -benchmem ./... -bench .` - メモリの使用状況も出力
            * `BenchmarkHoge  5000000    419 ns/op   115 B/op   3 allocs/op` - "使用容量(byte)" "メモリ割り当てオブジェクト数(?)"
        * `-cpu NUM` - `go test -cpu NUM ./... -bench .` - 使用するCPU数を指定（主にparalellテストで使用）
            * `BenchmarkHoge-N  5000000    238 ns/op   115 B/op   3 allocs/op` - "使用容量(byte)" "メモリ割り当てオブジェクト数(?)"
        * `-benchtime TIME文字列` - `go test -benchtime 1s ./... -bench .` - ベンチマークに費やす時間を指定



#### 他言語で同じ処理を書いてベンチマーク比較したい




#### ヘルプ
`go help ...`
* `test`
* `testfunc`
* `testflag`

