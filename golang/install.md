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
