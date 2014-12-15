# CoreOS

## これは何？
* サーバー用途向け軽量OS
* 仮想化コンテナを大規模に運用することに特化したLinuxOS, __Dockerに特化したOS__
    * "Docker Engine" - __アプリやミドル等、全てをDockerコンテナとしてインストールする__
        * 特定アプリケーション（「Apache」や「Nginx」といったサーバ）の依存関係をインストールするのではなく、アプリケーションをDockerコンテナ内に配置した後、CoreOSのインスタンス上にインストールする という形になる
        * なので、 __CoreOSにはaptやyumといった、Linuxで一般的となっているパッケージ更新ツールが含まれていない__
* メモリーが1Gバイト程度のマシンでも十分試せる軽量さが売りの __クラウドOS__
* アップデートが容易でセキュア。パッケージのアップデートよりも高速な、新バージョンへのアップデート
* ベースとなっているのは、GoogleのChrome OS
* クラスタリングを標準サポート(etcd)
* 分散システムをサポート(fleet)
* 多くのプラットフォームで動作する
    * 各種クラウドサービス(AWS, Google Compute Engine, Rackspace Cloud, Digital Ocean)
    * ベアメタル
    * 仮想環境(virtualbox, etc)
* 「RHEL は、常に機能を追加することで、その価値を高めるという方式を取ってきた。 しかし、CoreOS は機能を削ることで、価値を生み出している」

### 競合（？）
* __"Docker Engineを活用した分散環境のためのツール"__ という視点で
    * Kubernetes

### (2014/12) Docker批判 => Rocketの公開
* Dockerを“基本的に欠陥あり”と批判し、独自のコンテナランタイムRocketをローンした


## 核となる技術

### カーネル


### docker
* コンテナ
* アプリは基本的にDocker上で起動, ホストOSとの分離（これによって管理コスト削減）
    * ポータビリティ確保

#### Rocket
* ___Rocket___

### systemd

### etcd
* __クラスタ・オーケストレーションを行う__ 分散KVS

### fleet
* 分散システム
* fleet = etcd + systemd + job scheduling
* __複数ホストへサービスをデプロイ__
    * 管理コスト削減, 耐障害性

### cloud-config
* 設定ファイル = cloud-config (yaml) = `user-data.yml`
    * 先頭行が`#cloud-config`で始まるyamlファイルに起動時の設定を記述


## 起動、コンテナ起ち上げ
* CoreOSを起動する => SSHログインする(これBad？) => コンテナをbuild・runする


## vagrantで使う
* [coreos/coreos-vagrant](https://github.com/coreos/coreos-vagrant)をcloneしてきて、
* `config.rb`をsampleを参考に作成し、
* `user-data`をsampleを参考に作成し、
* vagrant upする

### 公式リポジトリのファイル構成
* `Vagrantfile`
* `config.rb.sample`
* `user-data.sample`


## 用途考察
* オンプレ、クラウド（、ローカル）に跨ったクラスタが構築できそう
    * オンプレのサービスがクラウドへ移行する際の選択肢として有用？考えられるロードマップは？
        * オンプレのCoreOS/コンテナ化 => クラウド移行
        * クラウド移行 => クラウドのCoreOS/コンテナ化 （まあこれかなぁ。。。）
        * 上記を段階踏まず1発でやる
* サービス規模拡大のスピードを阻害しない柔軟なインフラ拡大戦略
    * 少ない拡大作業コストで、スピーディに拡大が行えるか
    * マシン(オンプレorVM)のリソースの限界までコンテナを増やす => 限界が来たらマシンを増やす
* 自社サービスをコンテナ単位に細分化して負荷・リスクを軽減・分散する
    * 割りと見かけそうなパターンとして、`/home/COMPANY/SERVICE/SUB-SERVICEs`ってディレクトリ構成のサービスを運用してて、SUB-SERVICE AではSUB-SERVICE B, Cに依存があるがそれ以外とは依存関係が無い、という場合。
    * 余分なSUB-SERVICEを置きたくない => __SUB-SERVICE をコンテナ化__
        * SERVICE をマシン単位(= 1 CoreOS Machine)にする ってのも一考
    * 依存SUB-SERVICEの増減を自動検知出来れば最高(増加直後の初回アクセス時に該当SUB-SERVICEのコンテナを立ち上げる とか)
    * __etcdやfleetを何に使うか？（使わないか？）、左記2ツール以外で有用なものは無いか？もしっかり検討する必要がある__
    * (シナリオとワークフロー考えてローカルで試してみよう＆ソースに残しておこう)

### プラットフォーム
* CoreOS/Docker on AWS(EC2)
* CoreOS/Docker on Virtualbox + Vagrant
    * 開発環境構築用途として使う
    * [Vagrant + CoreOS + Dockerを利用した開発環境セットアップ](https://gist.github.com/yasushiyy/9923d68e4811d458fbe0)

### アーキテクチャ
* CoreOS/Docker + Consul
    * サービスディスカバリとしてConsulを組み合わせて使う
    * [DockerコンテナをConsulで管理する方法 - Qiita](http://qiita.com/foostan/items/a679ffcf3e20ff2f6032)
* CoreOS/Docker + MySQL
    * データのポータビリティをあげる戦略
    * 永続化が課題, Data Volume
        * ホストのvolumeと密結合になる
        * Data Volumeの変更はコンテナのイメージには含まれない
    * Docker公式には Data Volume Container を推奨している
        * "複数のコンテナ間で共有したい永続的なデータや非永続的なコンテナから参照したい永続的なデータは Data Volume Container と呼ばれるコンテナを作成して、そのコンテナからデータをマウントするのが良い"
    * => そこで  __Data-only container pattern__
        * [Docker でデータのポータビリティをあげ永続化しよう - Qiita](http://qiita.com/mopemope/items/b05ff7f603a5ad74bf55)
    * （ホントにポータビリティ実現できてるかイマイチ理解が怪しいのでもうちょい調べる）

