# CoreOS
[Documentation](https://coreos.com/docs/)

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
* クラスタリングを標準サポート(etcd)
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

### カーネル


### docker
* コンテナ
* アプリは基本的にDocker上で起動, ホストOSとの分離（これによって管理コスト削減）
    * ポータビリティ確保

#### Rocket
* ___Rocket___


### systemd
* __コンテナを1つのUnitとして起動させる__ のがCoreOS


### etcd
* __クラスタ・オーケストレーションを行う__ 分散KVS

### fleet
* クラスタノード管理
* fleet = etcd + systemd + job scheduling
* __複数ホストへサービスを分散デプロイ__
    * 管理コスト削減, 耐障害性
* フェールオーバー
* SSHログイン
* ジャーナルログの確認

### cloud-config
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
ざっくり試す流れはこう
* [coreos/coreos-vagrant](https://github.com/coreos/coreos-vagrant)をcloneしてきて、
* `config.rb`をsampleを参考に作成し、
* `user-data`をsampleを参考に作成し、
* vagrant upする

### 公式リポジトリのファイル構成
* `Vagrantfile`
* `config.rb.sample`
* `user-data.sample`

### 何をやってるかもう少し詳細に見てみる
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
          vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          vb.customize ["modifyvm", :id, "--uartmode1", serialFile]
        end
      end

      if $expose_docker_tcp
        config.vm.network "forwarded_port", guest: 2375, host: ($expose_docker_tcp + i - 1), auto_correct: true
      end

      config.vm.provider :vmware_fusion do |vb|
        vb.gui = $vb_gui
      end

      config.vm.provider :virtualbox do |vb|
        vb.gui = $vb_gui
        vb.memory = $vb_memory
        vb.cpus = $vb_cpus
      end

      #ip = "192.168.56.#{i+500}"
      ip = "172.17.8.#{i+200}"
      config.vm.network :private_network, ip: ip

      # Uncomment below to enable NFS for sharing the host machine into the coreos-vagrant VM.
      #config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']

      if File.exist?(CLOUD_CONFIG_PATH)
        config.vm.provision :file, :source => "#{CLOUD_CONFIG_PATH}", :destination => "/tmp/vagrantfile-user-data"
        config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      end

    end
  end
end
```




* __デフォルトでは `/vagrant` へのディレクトリ共有は行われない__





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
    * => そこで  __Data-only container pattern__
        * [Docker でデータのポータビリティをあげ永続化しよう - Qiita](http://qiita.com/mopemope/items/b05ff7f603a5ad74bf55)
    * （ホントにポータビリティ実現できてるかイマイチ理解が怪しいのでもうちょい調べる）
