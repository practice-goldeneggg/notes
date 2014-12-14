## テスト

### testingパッケージ


#### テスト(testing.T)


* テスト実行時に指定可能なフラグ  (ベンチマークで使用可能なものもあり)
    * `-v` - 詳細出力。`t.Log`等でのログ出力が有効になる
    * `go help testflag`で確認できる

#### ベンチマーク(testing.B)

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

```
usage: go test [-c] [-i] [build and test flags] [packages] [flags for test binary]
```

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


#### 標準出力の内容をテスト(ExampleXXX)


### テスト作成時のルール


### テストの実行
`go test <tests>`でテスト実行

