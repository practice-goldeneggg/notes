# CoreOS
[Documentation](https://coreos.com/docs/)

[An Introduction to CoreOS System Components | DigitalOcean](https://www.digitalocean.com/community/tutorials/an-introduction-to-coreos-system-components)


## これは何？
* サーバー用途向け軽量OS
* 仮想化コンテナを大規模に運用することに特化したLinuxOS, __Dockerに特化したOS__
    * "Docker Engine" - __アプリやミドル等、全てをDockerコンテナとしてインストールする__
        * 特定アプリケーション（「Apache」や「Nginx」といったサーバ）の依存関係をインストールするのではなく、アプリケーションをDockerコンテナ内に配置した後、CoreOSのインスタンス上にインストールする という形になる
        * なので、 __CoreOSにはaptやyumといった、Linuxで一般的となっているパッケージ更新ツールが含まれていない__
* メモリーが1Gバイト程度のマシンでも十分試せる軽量さが売りの __クラウドOS__
* アップデートが容易( __自動__ )でセキュア。パッケージのアップデートよりも高速な、新バージョンへのアップデート
    * 自動アップデートの無効化も可能
* ベースとなっているのは、GoogleのChrome OS
* 64bit CPUなら動く
* ファイルシステムは基本的にRead Only
* __クラスタリングを標準サポート__ (etcd)
    * クラスタ
    * ロール(役割)
    * ノード(nearly equal ホスト)
    * ユニット
* 分散システムをサポート(fleet)
* 多くのプラットフォームで動作する
    * 各種クラウドサービス(AWS, Google Compute Engine, Rackspace Cloud, Digital Ocean, さくらのクラウド)
    * ベアメタル
    * 仮想環境(virtualbox, etc)
* 「RHEL は、常に機能を追加することで、その価値を高めるという方式を取ってきた。 しかし、CoreOS は機能を削ることで、価値を生み出している」

>  A single service's code and all dependencies are packaged within a container that can be run on one or many CoreOS machines.
>  The lack of overhead allows you to gain density which means fewer machines to operate and a lower compute spend.
>  If you're currently running in the cloud, running a single CoreOS cluster on two different clouds or cloud + bare metal is supported and encouraged.'

### 競合（？）
* __"Docker Engineを活用した分散環境のためのツール"__ という視点で
    * Kubernetes

### (2014/12) Docker批判 => Rocketの公開
* Dockerを“基本的に欠陥あり”と批判し、独自のコンテナランタイムRocketをローンした


## 核となる技術
[Cluster Architectures](https://coreos.com/docs/cluster-management/setup/cluster-architectures/)

### docker
[Getting Started with docker](https://coreos.com/docs/launching-containers/building/getting-started-with-docker/)

* コンテナ
* アプリは基本的にDocker上で起動, ホストOSとの分離（これによって管理コスト削減）
    * ポータビリティ確保
* CoreOS立ち上げた時点でdockerサービスが起動してる
    * docker hubのコンテナ使いたかったら`docker pull`で即可能

#### Rocket
* ___Rocket___


### etcd - Cluster Management
[Distributed configuration data with etcd](https://coreos.com/blog/distributed-configuration-with-etcd/)

[Getting Started with etcd](https://coreos.com/docs/distributed-configuration/getting-started-with-etcd/)

* __クラスタ・オーケストレーションを行う__ 分散KVS
    * 設定の共有
    * service discovery("サービスの発見": 何処で・どんなサービス(コンテナ)が動いているかを知っている)
* APIを提供
    * [etcd API Documentation](https://coreos.com/docs/distributed-configuration/etcd-api/)
    * __実行中のコンテナ内からも叩ける__
        * CoreOS上で`ip address show`を叩き、NIC=docker0のIPへ向けて叩く
* Apache Zookeeperにインスパイアされている（らしい）
* ログの分散・複製には Raft アルゴリズムを採用している

#### Raft
[Raft](http://thesecretlivesofdata.com/raft/)

* consensus algorithm と呼ばれている
* 例えば更新操作は...
    * まず leader と呼ばれるnodeで受ける. commitはされない
    * leader から follwer と呼ばれるnodeへ内容が展開される 
    * leaderは follwer からの適用完了通知を待つ
    * 通知が来たらleader上で変更がcommitされる
    * leaderはcommitが完了したことをfollwerに通知する
    * これでいわゆる __consensus__ が完了, この一連の流れを __Log Replication__ と呼んでいる
        * こう見るとmysqlのレプリケーションぽくも見える
* leaderの選択 (Term1)
    * 選択の際のタイムアウト制御設定が2種類ある. __election timeout__ と __heartbeat timeout__
        * election timeout - follwer が candidate になるまで待つ時間
            * 150ms - 300ms の間
    * election timeout発生後、follwer が candidateになり、新しいelectionが始まる
    * candidateが自身へ投票を行い(Vote Count +1)、他のノードへ Request Vote メッセージを送る
    * このメッセージを受け取ったノードがまだ未voteであれば、送信元candidateへvoteする
    * voteが完了したらcandiateノードがelection timeoutをリセットする
    * candidateがvoteを過半数保持していればleaderになる
    * leaderになったノードは Append Entries メッセージをfollowerに送信し始める
        * このメッセージは heartbeat timeout で指定されたインターバルで送信される
    * このメッセージを受け取ったfollower達は、Append Entries メッセージをleaderへ返信する
    * このAppend Entriesのやりとりは、1つのfollowerが受信を止めるかcandidateになるまで続く
    * この一連のleader選択プロセスが完了したら、クラスタの状態（Raftでは __Term__ と呼んでいる）が更新された事になる
* leaderの停止, 再electionの発生 (Term2)
    * leaderノードが停止すると、別ノードがleaderになる
    * もし2つのノードが同時にcandidateになった場合,投票の分割が起こる
        * voteの過半数を必要とする事で、1つのノードしかleaderにはなれない という事を保証する
    * 2つのcandidateからの Request Vote が、それぞれ別のノードへ到達した場合, この時点で未到達のvoteはこれ以上何処にも受信されなくなる
        * Node CはAからの、DはBからのを受信 といったケース
    * こうなった場合は（仕方ないので）再投票が行われ、leaderを選出する
* log replication
    * leaderが決まると、他ノードへ全変更履歴をreplicateする事が必要になる
    * heartbeatに使用されていたのと同じ Append Entries メッセージを使って完了される
    * クライアントからleaderに変更処理が届くと...
        * 次回のheartbeatでは更新後の内容がfollwerに送信される
        * follwerの過半数がこれを受理したら、変更がcommitされる
        * クライアントに結果が返信される
    * Raftでは、ネットワークパーティションの表面?(face)で一貫性を保つことも可能
    * 例えば、現状1パーティションで構成されているノード群を2パーティションに分割してみる
        * 分割前leaderだったノードが所属しているパーティションでは、引き続きそのleaderが新leaderも務める
        * 逆に分割前leaderが所属していないパーティションでは、新たなleader選出プロセスが行われる
    * クライアントが1つ増えた場合
        * 一方のクライアントが __更新受信後にleaderがfollowerの過半数のheartbeatを得られないパーティション__ に変更処理を投げると、 __その変更はcommitされない__
        * もう一方で __leaderがfollowerの過半数のheartbeatを得られるパーティション__ に変更処理を投げると、 __その変更はcommitされる__
    * こうして各パーティションで内容齟齬がある状態で、逆にパーティションを統合してみる
        * commitされなかった側のleader は、より優先度の高いelection termを参照し,step downする
        * step down後に、そのleaderに所属するノードはcommitされてない変更をrollbackし、新leader(=commitされた側のleader)のlogに合わせる
        * こうしてクラスタ間のログ一貫性を保つ

### systemd
* __コンテナを1つのUnitとして起動させる__ のがCoreOS


### fleet - Launching Containers
[Process Management with Fleet](https://coreos.com/docs/quickstart/#process-management-with-fleet)

* クラスタノード群を管理する為の __init system__
    * systemdがベース
* fleet = etcd + systemd + job scheduling
* 複数ホストへサービスを分散デプロイ
    * 管理コスト削減, 耐障害性
* dockerコンテナのライフサイクル管理
* フェールオーバー
* SSHログイン
* ジャーナルログの確認

#### ロール
* worker

#### ユーティリティコマンド fleetctl
* `fleetctl load SERVICE`
* `fleetctl start SERVICE`
* `fleetctl status SERVICE`
* `fleetctl destroy SERVICE`
* `fleetctl list-machines`


### カーネル


### cloud-config
[Customize with Cloud-Config](https://coreos.com/docs/cluster-management/setup/cloudinit-cloud-config/)

* __クラスタ全体の__ 構成管理
* 設定ファイル = cloud-config (yaml) = `user-data.yml`
    * 先頭行が`#cloud-config`で始まるyamlファイルに起動時の設定を記述

```
# 先頭行は "#cloud-config" 必須
#cloud-config

# coreosクラスタ全体の設定開始宣言
coreos:
  # etcdの設定を行うセクション
  etcd:
    # discoveryをetcdに任せる為のトークン(ユニークなURL)をセットする。discoveryサーバ？を自前で用意することも出来るらしい
    # generate a new token for each unique cluster from https://discovery.etcd.io/new
    # WARNING: replace each time you 'vagrant destroy'
    #discovery: https://discovery.etcd.io/<token>
    addr: $public_ipv4:4001
    peer-addr: $public_ipv4:7001
  # fleetの設定を行うセクション
  fleet:
    public-ip: $public_ipv4
  # units(systemd)の設定を行うセクション
  units:
    # etcdを開始する
    - name: etcd.service
      command: start
    # fleetを開始する
    - name: fleet.service
      command: start
    # docker-tcpを開始する(リモートからのDockerAPI接続を可能にする)
    - name: docker-tcp.socket
      command: start
      enable: true
      content: |
        [Unit]
        Description=Docker Socket for the API

        [Socket]
        ListenStream=2375
        Service=docker.service
        BindIPv6Only=both

        [Install]
        WantedBy=sockets.target
```


## 起動、コンテナ起ち上げ
* CoreOSを起動する => SSHログインする(これBadって言ってる人いるけどそうなの？) => コンテナをbuild・runする


## vagrantで使う
[Vagrant](https://coreos.com/docs/running-coreos/platforms/vagrant/)


ざっくり試す流れはこう
* [coreos/coreos-vagrant](https://github.com/coreos/coreos-vagrant)をcloneしてきて、
* `config.rb`をsampleを参考に作成し、
* `user-data`([Cloud-Config](https://coreos.com/docs/running-coreos/platforms/vagrant/#cloud-config))をsampleを参考に作成し、
* vagrant upしcoreosを指定インスタンス数で起動

### 公式リポジトリのファイル構成
* `Vagrantfile`
* `config.rb.sample`
* `user-data.sample`

#### 何をやってるかもう少し詳細に見てみる
* Vagrantfile

```ruby
require 'fileutils'

Vagrant.require_version ">= 1.6.0"

CLOUD_CONFIG_PATH = File.join(File.dirname(__FILE__), "user-data")
CONFIG = File.join(File.dirname(__FILE__), "config.rb")

#-- デフォルト設定値, vagrant向け設定ファイル=config.rbが存在する場合は読み込んで上書かれる
# Defaults for config options defined in CONFIG
$num_instances = 1
$update_channel = "alpha"
$enable_serial_logging = false
$vb_gui = false
$vb_memory = 1024
$vb_cpus = 1

#-- 環境変数でのインスタンス数指定も対応するけどdeprecatedだよ
# Attempt to apply the deprecated environment variable NUM_INSTANCES to
# $num_instances while allowing config.rb to override it
if ENV["NUM_INSTANCES"].to_i > 0 && ENV["NUM_INSTANCES"]
  $num_instances = ENV["NUM_INSTANCES"].to_i
end

if File.exist?(CONFIG)
  require CONFIG
end

Vagrant.configure("2") do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = false

  config.vm.box = "coreos-%s" % $update_channel
  config.vm.box_version = ">= 308.0.1"
  config.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json" % $update_channel

  #-- vagrant up --provider vmware_fusion してvmware fusionを使用する場合に適用される
  config.vm.provider :vmware_fusion do |vb, override|
    override.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant_vmware_fusion.json" % $update_channel
  end

  config.vm.provider :virtualbox do |v|
    # On VirtualBox, we don't have guest additions or a functional vboxsf
    # in CoreOS, so tell Vagrant that so it can be smarter.
    v.check_guest_additions = false
    #-- これは何をやってるんだろうか？
    v.functional_vboxsf     = false
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  #-- 指定したインスタンス数分のCoreOS VMを立ち上げる
  (1..$num_instances).each do |i|
    config.vm.define vm_name = "core-%02d" % i do |config|
      config.vm.hostname = vm_name

      #-- シリアルロギング？が有効な場合の処理
      #-- （ファイルを介してUSB接続etcを行えるようにしたいってこと？）
      if $enable_serial_logging
        logdir = File.join(File.dirname(__FILE__), "log")
        FileUtils.mkdir_p(logdir)

        serialFile = File.join(logdir, "%s-serial.txt" % vm_name)
        FileUtils.touch(serialFile)

        config.vm.provider :vmware_fusion do |v, override|
          v.vmx["serial0.present"] = "TRUE"
          v.vmx["serial0.fileType"] = "file"
          v.vmx["serial0.fileName"] = serialFile
          v.vmx["serial0.tryNoRxLoss"] = "FALSE"
        end

        config.vm.provider :virtualbox do |vb, override|
          #-- "--uart<1-N> off|<I/O base> <IRQ>" 仮想マシンのシリアルポート設定
          vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          #-- "--uartmode<1-N> <arg>" ホストマシンに与えられた仮想シリアルポートを接続する方法を制御します。（事前に--uartXオプションで設定をしておく必要があり
          vb.customize ["modifyvm", :id, "--uartmode1", serialFile]
        end
      end

      if $expose_docker_tcp
        #-- ホストマシンからCore OS内のdocker apiアクセスを行うためのport forward
        config.vm.network "forwarded_port", guest: 2375, host: ($expose_docker_tcp + i - 1), auto_correct: true
      end

      config.vm.provider :vmware_fusion do |vb|
        vb.gui = $vb_gui
      end

      #-- スペックの設定
      config.vm.provider :virtualbox do |vb|
        vb.gui = $vb_gui
        vb.memory = $vb_memory
        vb.cpus = $vb_cpus
      end

      #-- ホストオンリーネットワーク設定
      #ip = "192.168.56.#{i+500}"
      ip = "172.17.8.#{i+200}"
      config.vm.network :private_network, ip: ip

      #-- NFSによるファイル共有(要permission)
      #--- "/vagrant" なディレクトリマウントはcoreos-vagrantでは行われない
      # Uncomment below to enable NFS for sharing the host machine into the coreos-vagrant VM.
      #--- HOST側は . から任意のディレクトリに書き換え
      #config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']

      if File.exist?(CLOUD_CONFIG_PATH)
        config.vm.provision :file, :source => "#{CLOUD_CONFIG_PATH}", :destination => "/tmp/vagrantfile-user-data"
        config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      end

    end
  end
end
```

### 試す - vagrantでcoreosインスタンスの立ち上げまで

* __デフォルトでは `/vagrant` へのディレクトリ共有は行われない__
* ひとまず `num_instances = 3`で試してみる

```
$ vagrant up
```

* 起動中のvm一覧を確認する
    * vm一覧の確認は`vagrant-global-status`プラグインで

```
$ vagrant plugin install vagrant-global-status
$ vagrant global-status -a
(なんか何も表示されないけど、なんでじゃろ)
```

* etcd, fleetプロセスが上がっている事を確認する

```
$ for v in $(vagrant status | grep core | awk '{print $1}'); do echo "=== $v" && vagrant ssh -c 'pgrep -l -f "(etcd|fleet)"' $v; done


=== core-01
593 etcd
599 fleetd
Connection to 127.0.0.1 closed.
=== core-02
844 etcd
850 fleetd
Connection to 127.0.0.1 closed.
=== core-03
853 etcd
858 fleetd
Connection to 127.0.0.1 closed.
```

### etcd で service discovery

#### service discovery とは
[Service Discovery with etcd](https://coreos.com/docs/quickstart/#service-discovery-with-etcd)

平たく言うと「利用可能なサービス（ここではノード）を通知する」為の仕組み

* 全ての実行中のCoreOSノードにetcdデータストアが展開される
* 各appコンテナはproxyコンテナに自分自身の存在を知らせる
    * proxyコンテナはトラフィックを受信すべきマシンの自動検知を担う
* アプリにserivice discoveryの仕組みを構築する事により、マシン追加やサービスのスケールをシームレスに(違和感なく)実現する
* https://discovery.etcd.io/new にアクセスしてトークンを取得し、`cloud-config`に設定する
    * ___vagrant使用時は、destroyの度にトークンを取り直す必要あり___
* CoreOSマシン内から(ローカルの)etcd APIの動作確認
    * `curl -L http://127.0.0.1:4001/v1/keys/message -d value="Hello world"` `{"action":"set","key":"/message","prevValue":"Hello world","value":"Hello world","index":12426}`
    * [etcd API Documentation](https://coreos.com/docs/distributed-configuration/etcd-api/)
* コンテナ内から母艦CoreOSへのetcd API実行
    * [Reading and Writing from Inside a Container](https://coreos.com/docs/distributed-configuration/getting-started-with-etcd/#reading-and-writing-from-inside-a-container)
    * `ip address show`で __NIC=docker0__ のIPアドレスへ向けて叩く

    ```
    4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN
        link/ether 56:84:7a:fe:97:99 brd ff:ff:ff:ff:ff:ff
        inet 10.1.42.1/16 scope global docker0
           valid_lft forever preferred_lft forever
        inet6 fe80::5484:7aff:fefe:9799/64 scope link
           valid_lft forever preferred_lft forever
    ```
    ```
    curl -L http://10.1.42.1:4001/v2/keys/
    {"action":"get","node":{"key":"/","dir":true,"nodes":[{"key":"/coreos.com","dir":true,"modifiedIndex":9,"createdIndex":9},{"key":"/message","value":"Hello world","modifiedIndex":12426,"createdIndex":12426}]}}
    ```

#### 試す - (小さい)クラスタを
[Small Cluster](https://coreos.com/docs/cluster-management/setup/cluster-architectures/#small-cluster)

[Clustering Machines](https://coreos.com/docs/cluster-management/setup/cluster-discovery/)

[Clustering CoreOS with Vagrant](https://coreos.com/blog/coreos-clustering-with-vagrant/)


* __3 - 9 ノード__ が目安, とりあえず3ノードで
* https://discovery.etcd.io で新規tokenを取得する
    * etcdはクラスタリングに __etcd discovery URL__ を使っていて、このURL内に一意なtokenを含んでいる

```
$ curl https://discovery.etcd.io/new
https://discovery.etcd.io/HASH
```

* `cloud-config`に取得したtokenを記述する

```
$ vi user-data

coreos:
  etcd:
    discovery: https://discovery.etcd.io/HASH
```

* vagrant起動or再起動する

```
$ vagrant reload --provision
```

* 認証エージェントにvagrant用の鍵を追加し、__ホストマシン=>任意のcoreos vm__, __coreos vm X => coreos vm Y__ 間のssh接続をノーパスで行えるようにする

```
$ ssh-add ~/.vagrant.d/insecure_private_key
Identity added: /Users/CoreOS/.vagrant.d/insecure_private_key (/Users/CoreOS/.vagrant.d/insecure_private_key)

$ vagrant ssh core-01 -- -A
CoreOS alpha (540.0.0)
core@core-01 ~ $
```

* クラスタリングできてるか確認

```
core@core-01 ~ $ fleetctl list-machines
MACHINE         IP              METADATA
2a4...         172.17.8.201    -
2a5...         172.17.8.202    -
2a6...         172.17.8.203    -
```

* こんなエラーが出る時がある。 __`vagrant destroy`した後に https://discovery.etcd.io/new のトークンを再取得しないとダメなんだがそれが漏れてる__ というケースがほとんど

```
core@core-01 ~ $ fleetctl list-machines
2015/01/19 03:20:34 INFO client.go:291: Failed getting response from http://127.0.0.1:4001/: dial tcp 127.0.0.1:4001: connection refused
2015/01/19 03:20:34 ERROR client.go:213: Unable to get result for {Get /_coreos.com/fleet/machines}, retrying in 100ms
2015/01/19 03:20:34 INFO client.go:291: Failed getting response from http://127.0.0.1:4001/: dial tcp 127.0.0.1:4001: connection refused
2015/01/19 03:20:34 ERROR client.go:213: Unable to get result for {Get /_coreos.com/fleet/machines}, retrying in 200ms
```

* データのset,getが各ノード間で連携されている事を確認

```
# どこぞのホストで
$ curl -L http://127.0.0.1:4001/v1/keys/message -d value="Aho world"
{"action":"set","key":"/message","value":"Aho world","newKey":true,"index":106}

# 各ホストで

```



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
        * コード書く => CIする のイテレーションをどう回すんだろうか？（どういう実作業が発生するのだろうか？という意味で）
            * コード書く => docker build(差分) => docker run
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
    * => そこで __Data-only container pattern__
        * [Docker でデータのポータビリティをあげ永続化しよう - Qiita](http://qiita.com/mopemope/items/b05ff7f603a5ad74bf55)
    * （ホントにポータビリティ実現できてるかイマイチ理解が怪しいのでもうちょい調べる）

### 課題
* etcd自体の稼働監視と障害復旧
* fleet自体の稼働監視と障害復旧
* cloud-config作ってvagrant upしてvagrant ssh CLUSTER する流れがビミョーにメンドイ
    * vagrant destroy の後にdiscovery token再取得しないとダメなところとか
