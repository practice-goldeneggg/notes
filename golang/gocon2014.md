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
