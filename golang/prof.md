## プロファイリング
[Profiling Go Programs - The Go Blog](https://blog.golang.org/profiling-go-programs)

### ブクマ


### 標準のプロファイラ(runtime/pprof)
* [pprof - The Go Programming Language](http://golang.org/pkg/runtime/pprof/)
* [Description of testing flags](http://golang.org/cmd/go/#hdr-Description_of_testing_flags)
    * `-XXXprofile FILENAME` - 系オプションで指定可能, 結果をFILENAMEファイルに出力する
        * `-blockprofile` - "プロファイラはプログラムがブロックされているnナノ秒ごとに1つのブロックイベントとして平均的にサンプルするように努めます"
            * `-blockprofilerate NUM` - runtime.SetBlockProfileRateをNUM(ナノ秒)を指定して呼び出すことにより、goroutineのブロックプロファイルに供給する詳細度を制御する
        * `-coverprofile` - カバレッジ
            * `-cover` - カバレッジを有効にする(`-coverprofile`を指定すると自動的に有効になる)
            * `-covermode <set|count|atomic>` - カバレッジ解析モードを設定
            * `-coverpkg <pkg1,pkg2,pkgn>` - 与えられたパッケージリストに対し、各テストでカバレッジ解析を適用
        * `-cpuprofile` - CPU
        * `-memprofile` - メモリ
            * `-memprofilerate NUM` - より正確な（ただしコストの掛かる）メモリプロファイルを有効にする
    * テストバイナリ実行時にこれらのオプションを指定する際は`-test.`というprefixを付ける必要あり

#### cpuprofile

* 既存Appのmain関数に下記のようなコードを埋め込む
    * `pprof.StartCPUProfile(file)` - cpuプロファイル結果をfileに出力する プロファイルを開始する
    * `pprof.StopCPUProfile()` - cpuプロファイルを停止する(通常は`defer`で呼ぶ)

```go
var cpuprofile = flag.String("cpuprofile", "", "write cpu profile to file")

func main() {
    flag.Parse()
    if *cpuprofile != "" {
        f, err := os.Create(*cpuprofile)
        if err != nil {
            log.Fatal(err)
        }
        pprof.StartCPUProfile(f)
        defer pprof.StopCPUProfile()
    }
```

* `-cpuprofile=PROFILE_FILE_NAME`オプション付きで実行する
* `go tool pprof PROFILE_FILE_NAME`コマンドで出力したプロファイル結果を元にpprofを起動する

```
$ go tool pprof hoge.prof

Entering interactive mode (type "help" for commands)
(pprof) help

 Commands:
   cmd [n] [--cum] [focus_regex]* [-ignore_regex]*
       Produce a text report with the top n entries.
       Include samples matching focus_regex, and exclude ignore_regex.
       Add --cum to sort using cumulative data.
       Available commands:
         callgrind    Outputs a graph in callgrind format
          :
          :

# 負荷top10を(降順で)表示
(pprof) top10
Total: 2525 samples
     298  11.8%  11.8%      345  13.7% runtime.mapaccess1_fast64
     268  10.6%  22.4%     2124  84.1% main.FindLoops
     251   9.9%  32.4%      451  17.9% scanblock
     178   7.0%  39.4%      351  13.9% hash_insert
     131   5.2%  44.6%      158   6.3% sweepspan
     119   4.7%  49.3%      350  13.9% main.DFS
      96   3.8%  53.1%       98   3.9% flushptrbuf
      95   3.8%  56.9%       95   3.8% runtime.aeshash64
      95   3.8%  60.6%      101   4.0% runtime.settype_flush
      88   3.5%  64.1%      988  39.1% runtime.mallocgc

(pprof) top5 -cum

(pprof) web

(pprof) web mapaccess1

(pprof) list <FUNC NAME>

```

#### memprofile

* 既存Appのmain関数に下記のようなコードを埋め込む
    * `pprof.WriteHeapProfile(file)` - memプロファイル結果をfileに出力する プロファイルを開始する

```go
var memprofile = flag.String("memprofile", "", "write mem profile to file")

func main() {
    flag.Parse()
    if *memprofile != "" {
        f, err := os.Create(*memprofile)
        if err != nil {
            log.Fatal(err)
        }
        pprof.WriteHeapProfile(f)  // cpuのようにStart...って名前になっていない
        defer f.Close()  // defer内でcloseする
    }
```

* `-cpuprofile=PROFILE_FILE_NAME`オプション付きで実行する
* `go tool pprof PROFILE_FILE_NAME`コマンドで出力したプロファイル結果を元にpprofを起動する



### サーバーサイド(net/http/pprof)

