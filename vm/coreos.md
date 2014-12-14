# CoreOS

## これは何？
* サーバー用途向け軽量OS
* 仮想化コンテナを大規模に運用することに特化したLinuxOS, __Dockerに特化したOS__
    * "Docker Engine"
    * __特定アプリケーション（「Apache」や「Nginx」といったサーバ）の依存関係をインストールするのではなく、アプリケーションをDockerコンテナ内に配置した後、CoreOSのインスタンス上にインストールする__ という形になる
        * なので、 __CoreOSにはaptやyumといった、Linuxで一般的となっているパッケージ更新ツールが含まれていない__
* メモリーが1Gバイト程度のマシンでも十分試せる軽量さが売りの __クラウドOS__
* ベースとなっているのは、GoogleのChrome OS
* クラスタを標準サポート(etcd)
* 分散システムをサポート(fleet)
* 多くのプラットフォームで動作する
    * 各種クラウドサービス(AWS, Google Compute Engine, Rackspace Cloud)
    * ベアメタル
    * 仮想環境(virtualbox, etc)

### 競合（？）
* __"Docker Engineを活用した分散環境のためのツール"__ という視点で
    * Kubernetes


## 構成要素
* カーネル
* systemd
* Docker

## 核となる技術

### docker
* コンテナ
* アプリは基本的にDocker上で起動, ホストOSとの分離（これによって管理コスト削減）
    * ポータビリティ確保

### etcd
* クラスタ・オーケストレーションを行う分散KVS

### fleet
* 分散システム
* fleet = etcd + systemd + job scheduling
* 複数ホストへサービスをデプロイ, 管理コスト削減, 耐障害性


## vagrantで使う
* [coreos/coreos-vagrant](https://github.com/coreos/coreos-vagrant)をcloneしてきて、
* `config.rb`をsampleを参考に作成し、
* `user-data`をsampleを参考に作成し、
* vagrant upする

### 公式リポジトリのファイル構成
* `Vagrantfile`
* `config.rb.sample`
* `user-date.sample`
