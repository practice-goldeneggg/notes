gocon
=====


プレゼン
========

キーノート以外は一人15-20分程度(質疑の時間も含む)を考えていますが、柔軟に時間調整します。
また発表は掲載順ではないので気をつけてください。
(Presentation order is random but for Keynote)

Keynote1: Rob Pike (@rob_pike) (45min)
--------------------------------------------
* Go成功の最大の理由＝シンプルさ
* Java, C, js, PHP等はお互いの言語を積極的に借りてきている->一つの言語への収束
* サピア＝ウォーフの仮説
    * [サピア＝ウォーフの仮説 - Wikipedia](http://ja.wikipedia.org/wiki/%E3%82%B5%E3%83%94%E3%82%A2%EF%BC%9D%E3%82%A6%E3%82%A9%E3%83%BC%E3%83%95%E3%81%AE%E4%BB%AE%E8%AA%AC)
* 言語が全て収束すればプログラマの考え方は皆おなじになる -> がしかし考え方が異なるという事は良いこと
* そしてこれらの言語は収束しつつ機能追加することで複雑さを増してきた
* __Goは違う__
     * 必要な機能（だけ）を取り込む
     * でもシンプルに
* 機能が多すぎると「どの機能を使うべきか」の選択に時間を取られる
    * 複雑な言語を使っているという理由だけで可読性が落ちる
    * 可読性は信頼性
* 表現力
    * APLのコードが読みやすいか？違うでしょう
    * 単純なものは単純なプリミティブで実装できるべき(実行コストも上がるし、パフォーマンスも予測しづらくなる)
* 機能のための機能はいらない「時間を埋める」機能が必要, 単純に動作する単純な機能 が必要
* "スケーラブルなクラウドソフトウェアのために設計された綺麗な手続き型言語"
* 実際には複雑だけれど単純だと感じる のがGo, 要素はお互いにシームレスに調和し、驚きが合ってはいけない
* 単純に書けるのはGCのおかげ
* gorouteineのシンプルさ
    * スタックサイズがない
    * returnやステータスがない
    * 管理機構が無い
    * ID がない
    * __"g" "o" " " を押すだけで起動できる__
* 複雑さの隠蔽
    * 隠された複雑さ に踏み込みたくなるようなケースがもしあるようなら、それはGoを使うべきではないケースって事なのだろうな #gocon



Keynote2: @fumitoshi_ukai (45min)
--------------------------------------------
* Readability Reviewの話
* 複雑さを隠そうとしてかえって複雑になるってケースが（JavaやC++だと）よく見かける #gocon
* インデントどうするか？という議論は本質的ではない。だったらgoがツールを提供して任せてしまえば良い=gogmt
* regexpのCompileとMustCompile、(errorが返ってこない)MustCompileは初期化の時に使いましょう
* deferとClose、readのCloseは "defer r.Close()" で良いが、writeはClose()のerrorもちゃんとハンドリングすべき
* エラーに色んな情報を含めたい場合は、エラー情報保持用structを作って管理すると良い。そのstructにエラーメッセージを返すメソッドを定義 #gocon
* 名前を簡潔に 
* 30 * time.Second の 30 は定数なので、この式ではDuration型として扱われる
* 形が分かってる時にreflect使うのはよろしくない
* コメントにAPIの使い方を書くぐらいならExampleテストを書けば良い（実際に使えるExampleとして使える）
* packageコメント もちゃんと書きましょう
    * コメントは対象としているものの名前から始まる文にしよう
    * 書いたコメントはgodocで確認
* "package util" って名前も良くないし、そういう用途のpackageが作られること自体良くない。
* 引数などはinterfaceの方が組み合わせしやすいしテストもしやすい
* panicは使うな, errorを使え
* ゴルーチンを起動することを利用者に求めるようなAPIは良くない。呼び出し側が同期/非同期を選べる設計にしておくべき（つまりAPIは同期で書こう） #gocon



@moriyoshit
-----------
TBA




@methane
----------------------------

:タイトル:
 Why Go is so slow.

:内容（予定）:

 * プロファイラーの使い方
 * 速いGoコードの書き方
 * 将来のGoに期待すること

* Explicit is better than implicit
* Simple is better than complex
* runtime/pprof
    * net/http/pprof - デーモンとかに向いてる
    * github.com//davecheney/profile - CLIに向いてる
* サンプリングする処理はgoのパッケージに入ってる。解析するプログラムはperlのCLI(<= 1.3) perftoolsのpprof
    * 1.4からはGoで再実装されてる
* Hello Wolrdのサンプル, net/http使うなら __Content-Typeをセットしろ__ ___セットしないと遅くなるよ___
* DEMO - ( __macだとpprof使えない__ のでアクティビティモニタのサンプル収集で)
* pprofなら ... `go tool pprof APPNAME URL`
* pprofの使い方
    * pprofコンソールに入る
    * `top`コマンド
    * `weblist` - `list`(アセンブラの詳細確認するやつ)をブラウザで見れる __1.4から__
* で、遅い理由って何が多い？
    * GC
        * GODEBUG=gctrace=1
        * sync.Pool
    * メモリコピー
        * string にするか []byte にするか注意しよう, グローバル変数なら[]byte (って言ってたような？)
        * `str[:1:1]` - :N:N って書き方（あとで調べる) 1.3から
    * 関数呼び出し
        * インライン展開で回避できる（かも） 葉の関数（leaf function）
        * ループ丸ごと関数にした vs ループの一部処理を別関数にした の比較 (あとで資料見て確認)




Greg 
----------------------------

:タイトル:
 NSQ-Centric Architecture

:内容（予定）:

 * Using NSQ to develop a distributed chat app
 * Go + React + NSQ dream team

* 




@songmu
----------------------------

:タイトル:
 mackerel-agent徹底解説

:内容:
 今や世界中のサーバーで稼働しているmackerel-agentについて解説します。主に以下の様な内容を取り上げます。

 - シグナルハンドリング
     - ホスト情報のリロード
     - graceful shutdown
 - メトリクス送信処理
     - 毎分の処理と制御
     - メトリクス再送処理
     - メトリクスをまとめて送る

* agentがGo製で、社内や顧客のメトリクス集計対象サーバに入れて常駐起動させてる
* Goは常駐プロセスを書くのに向いている
* SYGINTはwinで動かないので os.Interrupt を使う
    * os.Interrupt を使おう、って話が出て自分のソースにあるシグナルハンドル処理調べたらガッツリ SIGINT って書いてた...mackerelさんアザス


@qt_luigi
------------------------------

:タイトル:
 Golang JP Community

:内容:
 タイトルには二つの意味を込めました。私が管理メンバーをさせて頂いている「Golang JP Google+ コミュニティ」と、来日されたRob Pike氏への報告も兼ねて「日本のGoのコミュニティ活動」について、ご紹介させて頂きます。

* Top 5 community
    * Go+
    * GDK Korea Golang
    * Golang JP は5位



@sinmetal
----------------------
:タイトル:
 App Engine for Golang Performance

:内容:
 AppEngine for Golang Performance, Managed VMs for Golangについて話します。

* Bad AppEngine
    * ver 1.2
    * GOMAXPROCS=1
    * Eternal Beta...





@ironzeb
---------------------

* gengo.com の中の人




@y_matsuwitter
--------------
:タイトル:
 Golang @ISUCON

:内容（予定）:

 * 初参戦したISUCONでのGolangを使った話
 * 主にプロセスキャッシュや複数インスタンスでの分散処理について


* 高速なWebサーバ
    * CPU, Memory, Data Transfer
* Goを選んだ理由 = __短時間で__ 性能を上げる という用途にGoのシンプルさが最適だった
    * 並行処理でのメモリの扱いが楽
    * バイナリを置けば動く=デプロイが楽
* Race conditionに気をつける (goroutineをまたいだ共有値の読み書き) 適切なLock
* atomic.Value (>= 1.4) がプロセスキャッシュ?に有用
* 重複処理を避ける（同じ画像を取得するリクエストが複数飛んできて後者を効率的に処理（スルー）したい） = sync.Cond が便利



Lightning Talk
==============

LTは1人5分程度を目安でお願いします。
なお、質疑の時間は予定していません。
また発表は掲載順ではないので気をつけてください。
(Presentation order is random but for Keynote)

.. 追記例
.. 
.. @hogehoge
.. ________
.. タイトルや概要、資料へのリンク



@nuki_pon
----------------------
Tシャツの話




@tkak
----------------------
:タイトル:
 Terraformのpluginについて

:内容:
 http://tkak.hatenablog.com/entry/2014/11/07/074044




ainoya
--------------------------------------------
:タイトル:
 Goでビルドパイプラインツールを書いた話

:内容:
 ツール開発時のtipsや，ビルドジョブ間のメッセージパッシングをgoroutineで実装したことなどについて話したいです．(https://github.com/walter-cd/walter)



@yuroyoro
-------------
:タイトル:
 go/parser, go/astの話

:内容:
 goの抽象構文木で遊ぶ話




@cubicdaiya
------------------------
:タイトル:
 nginx-build

:内容:
 Goでnginxのビルド自動化ツールを書いた話



@yuya_takeyama
----------------------
「Go におけるユニットテストについて」 or 「Go でコマンドラインツールを作ることについて」




@imkira85
--------------------
:タイトル:
 分散型システムをつくろう

:内容:
 etcd/consul/consistentを使ったものについて話します。



