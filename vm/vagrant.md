# Vagrant

===

## Getting Started

### 前提条件
* Virtualbox入れとく(macで試すならとりあえず)
* gem(ruby)入れとく
  * rbenv

### インストール
* ~~1.0系を入れる~~
* __最新の 1.4系 を入れる__　※packageインストールに変わってる
  * [Vagrant - Downloads](http://downloads.vagrantup.com/)
  * gemで入れた古いverが存在する場合、先に gem uninstall しとく
    * rbenvで管理してる場合、~/.rbenv/shims から手動削除が必要かも
    * ruby-default-gems使用時は、default listからも削除しとく

### ひな形のセットアップ, boxの追加(vagrant box add)
例として、CentOS6.4 x86_64 minimam を使用

```
$ vagrant box add centos64_86_min http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-x86_64-v20130427.box

$ ls ~/.vagrant.d/boxes
centos64_86_min
```

* 追加したboxの一覧は vagrant box list で確認可能

```
$ vagrant box list
Berkshelf-CentOS-6.4-x86_64 (virtualbox)
centos64_86_min             (virtualbox)
centos65_86                 (virtualbox)
debian_wheezy_amd64         (virtualbox)
ubuntu_1304_amd64_noguest   (virtualbox)
```

### 仮想マシン初期化(vagrant init)

```
$ mkdir ~/vagrant/centos64_86_min
$ cd ~/vagrant/centos64_86_min

$ vagrant init centos64_86_min
```

### 設定変更(Vagrantfile)
Vagrantfileの変更を適宜実施

```ruby
Vagrant.configure("2") do |config|


  config.vm.hostname = "vmchef1" # ★ホスト名
  config.vm.network :private_network, ip: "192.168.56.21" # ★ホストオンリーネットワークでIP固定

end
```

### 起動(vagrant up)

```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
[default] Importing base box 'centos64_86_min'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] Booting VM...
[default] Waiting for VM to boot. This can take a few minutes.
[default] VM booted and ready for use!
[default] Setting hostname...
[default] Configuring and enabling network interfaces...
[default] Mounting shared folders...
[default] -- /vagrant
```

* 起動が激遅いときがちょくちょくある。この場合、一旦 vagrant destroy してから再度 vagrant up してみる
* 詳細なログが欲しい場合は --debug オプションを使用する

### sshログイン

```
$ vagrant ssh  # もしくは ssh vagrant@192.168.56.21
Last login: Wed Mar 13 05:07:52 2013 from 10.0.2.2

[vagrant@vmcheftry ~]$ ifconfig
eth0      Link encap:Ethernet  HWaddr 08:00:27:41:F0:43
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe41:f043/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:373 errors:0 dropped:0 overruns:0 frame:0
          TX packets:234 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:38000 (37.1 KiB)  TX bytes:33225 (32.4 KiB)

eth1      Link encap:Ethernet  HWaddr 08:00:27:9B:54:7D
          inet addr:192.168.56.21  Bcast:192.168.56.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe9b:547d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:12 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 b)  TX bytes:720 (720.0 b)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

[vagrant@localhost ~]$ ping yahoo.jp　★外部とのネットワーク接続有効か確認
PING yahoo.jp (183.79.23.196) 56(84) bytes of data.
64 bytes from yjpn200.mobile.vip.kks.yahoo.co.jp (183.79.23.196): icmp_seq=1 ttl=63 time=19.7 ms
64 bytes from yjpn200.mobile.vip.kks.yahoo.co.jp (183.79.23.196): icmp_seq=2 ttl=63 time=19.5 ms

[vagrant@localhost ~]$ ls -la
合計 36
drwx------. 3 vagrant vagrant 4096  3月 13 06:08 2013 .
drwxr-xr-x. 3 root    root    4096  7月 10 22:14 2012 ..
-rw-------  1 vagrant vagrant  136  3月 13 05:15 2013 .bash_history
-rw-r--r--. 1 vagrant vagrant   18  5月 10 20:45 2012 .bash_logout
-rw-r--r--. 1 vagrant vagrant  176  5月 10 20:45 2012 .bash_profile
-rw-r--r--. 1 vagrant vagrant  124  5月 10 20:45 2012 .bashrc
drwx------  2 vagrant root    4096  7月 10 22:19 2012 .ssh
-rw-------  1 vagrant vagrant    7  7月 10 22:14 2012 .vbox_version
-rw-r--r--  1 vagrant vagrant 1506  7月 10 22:14 2012 postinstall.sh　★TODO調査

[vagrant@localhost ~]$ ls -la /vargrant
total 12
drwxr-xr-x   1 vagrant vagrant  170 Apr 10 10:21 .
dr-xr-xr-x. 24 root    root    4096 Apr 10 17:25 ..
drwxr-xr-x   1 vagrant vagrant  272 Apr 11 11:11 chef-repo
-rw-r--r--   1 vagrant vagrant   61 Apr 10 09:54 .vagrant
-rw-r--r--   1 vagrant vagrant 4073 Apr 10 09:53 Vagrantfile

★↑ ホストOS側で作成した環境（chefのcookbookとかも）がゲストOSの /vagrant に展開されている（これでゲストOSからcookbookが見えるようになる）
```

### リロード(vagrant reload)

* guest側のrestart。Vagrantfileを修正した後とか

```
$ vagrant reload
```

### 停止(vagrant halt)

```
$ vagrant halt
[default] Attempting graceful shutdown of VM...
```

### 一時停止(vagrant suspend)

```
$ vagrant suspend
```

### 破棄(vagrant destroy)

```
$ vagrant destroy
[default] Attempting graceful shutdown of VM...
```

### パッケージング(vagrant package)

* 作成したVMからパッケージ(box)を作成する。
  * virtualbox用のbase boxを作成した場合は --base オプションを指定
* 作成したパッケージは第三者に共有するなり配布するなりし、vagrant box add hoge.box で全く同じ環境のVMを導入可能

```
$ vagrant package
```

### vmの状態確認(vagrant status)
* 作成したvmの一覧、それらのvmが起動中か停止中か といった状態の表示
* Vagrantfileを配置しているディレクトリ下で実行する

```
$ vagrant status

Current machine states:

wstd1                     not created (virtualbox)
tdcoll1                   not created (virtualbox)
es1                       not created (virtualbox)
```

### vmにプロビジョニングを行う(vagrant provision)

* chef等のプロビジョニングツールでのprovisioningを実行
* "vagrant provision" OR "vagrant reload --provision" で実行される
* up時も(デフォルトは)実行される
  * up時にプロビジョニングを実行したくない場合は --no-provision オプションを付加する
* 詳細は Vagrantfileの設定もろもろ の項で

```
$ vagrant provision
```

===

## Vagrantfileの設定もろもろ

### ネットワーク関連
* forwarded_port - hostからguestのポートにフォワーディングする
  * guest側にapache立てて検証したい場合とか、いちいちprivate_network(後述)設定するより楽ちんかも

```
config.vm.network :forwarded_port, host: 4567, guest: 80

config.vm.network :forwarded_port, host: 8080, guest: 80, host_ip: "192.168.56.51"

config.vm.network :forwarded_port, guest: 2003, host: 12003, protocol: 'tcp' # デフォルトはtcp

config.vm.network :forwarded_port, guest: 2003, host: 12003, protocol: 'ucp'

config.vm.network :forwarded_port, host: 4567, guest: 80, auto_correct: true # ポートの衝突検出・自動訂正
```

* private_network - いわゆるホストオンリーネットワークの設定
  * ※ "virtualbox" というprefixは、virtualbox専用のオプションの事らしい

```
config.vm.network "private_network", ip: "192.168.50.4" # 静的IPのassign

config.vm.network "private_network", ip: "192.168.50.4", virtualbox__intnet: true # virtualboxでのみ使用可能。ホストオンリーではなくinternal network(bridgeと似てる仕組み、外部インターネットと直接接続)にする

config.vm.network "private_network", ip: "192.168.50.4", virtualbox__intnet: "mynetwork" # internal networkに名前を付ける
```

* public_network - そのまま、IPの割り当てはDHCPで行われる

```
config.vm.network "public_network"

config.vm.network "public_network", :bridge => 'en1: Wi-Fi (AirPort)' # ネットワークインタフェースの指定
```

### ディレクトリの同期

* guest側の "/vagrant" ディレクトリは、host側の Vagrantfile配置ディレクトリと同期する
* "synced_folder [hostのパス] [guestのパス]" で明示的な同期指定も可能

```
config.vm.synced_folder "src/", "/srv/website"

config.vm.synced_folder "src/", "/srv/website", disabled: true # 使用不可にする場合disabled

config.vm.synced_folder "src/", "/srv/website", owner: "root", group: "root
```

* NFSもサポートしてる


### 複数VM起動
* "config.vm.define" を複数ブロック記述する形式で可能

```
% vi Vagrantfile

  config.vm.base = "centos64_86_min"

  config.vm.define :test1 do |test1|
      test1.vm.hostname = "vmtest1"
      test1.vm.network :private_network, ip: "192.168.56.21"

      # for VirtualBox
      test1.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", 1024]
      end
  end

  config.vm.define :db1 do |db1|
      db1.vm.hostname = "vmdb1"
      db1.vm.network :private_network, ip: "192.168.56.22"

      # for VirtualBox
      db1.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", 1024]
      end
  end
```

```
% vagrant up

% vagrant ssh test1

[vagrant@vmtest1 ~]$ ifconfig
eth1      Link encap:Ethernet  HWaddr 08:00:27:CA:00:FF
          inet addr:192.168.56.21  Bcast:192.168.56.255  Mask:255.255.255.0

% vagrant ssh db1 
[vagrant@vmdb1 ~]$ ifconfig
eth1      Link encap:Ethernet  HWaddr 08:00:27:CA:00:FF
          inet addr:192.168.56.22  Bcast:192.168.56.255  Mask:255.255.255.0
```

* 個別のvmを指定してupする → vagrant up [vm名] でOK
  * vm名 は正規表現指定も可能

```
% vagrant up db1

% vagrant up /slave[0-9]/
```

* primary: true すると、primary machineになる

```
config.vm.define "web", primary: true do |web|
  # ...
end
```

### provider
仮想環境のプロバイダ(≒プラットフォーム)

* 起動時に指定する

```
$ vagrant up --provider=vmware_fusion
$ vagrant up --provider=aws
```

* Vagrantfileで各種providerパラメータのカスタマイズが可能

```
config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
end
```

* providerパラメータ以外の設定値は override が可能

```
config.vm.box = "precise64"

config.vm.provider "vmware_fusion" do |v, override|
  override.vm.box = "precise64_fusion"
end
```

* 同じmachineを複数のproviderで起動することは出来ない（が、将来的にこの制限は撤廃する予定らしい）


#### Virtualbox
* [Creating a Base Box - VirtualBox Provider - Vagrant Documentation](http://docs.vagrantup.com/v2/virtualbox/boxes.html)
* [Creating a Base Box - Vagrant Documentation](http://docs.vagrantup.com/v2/boxes/base.html)

* vmをパッケージング(して配布とか)出来る。→ "base box" ってやつ
  * vagrant box add hoge なコマンドで、ネット上に転がってるbox同様の環境構築が可能
* base boxの構成要素
  * Package Manager
  * SSH
      * vagrant ユーザーで接続できること
  * PuppetとかChefのようなプロビジョニングツール（必須ではない）
* base box作成対象のvmでの作業ガイドライン
  * **virtualbox guest additions**を追加でインストールしておく必要あり
  * vagrant ユーザーでssh出来る状態にしておく
  * rootのパスワードは "vagrant" を設定しておく
  * vagrantユーザーがパスワード無しでsudo出来るようにしておく
  * sshサーバーでDNSルックアップを抑止（UseDNSをnoにする）

```
# guest additionsのインストール、Ubuntuの例

$ sudo apt-get install linux-headers-$(uname -r) build-essential
$ sudo mount /dev/cdrom /mnt/cdrom
$ sudo sh /media/cdrom/VBoxLinuxAdditions.run
```

```
# "Virtualbox上での" VM名の確認
$ VBoxManage list vms
"hoge" {xxxx-yyyy-zzzz}

# 確認したVM名を指定してbase box作成
$ vagrant halt # 一旦停止(しろと書いてあるWeb情報多し)
$ vagrant package --base hoge --output /path/to/the/new.box

# base boxをbox addして初期化・起動するまで
$ vagrant box add my-box /path/to/the/new.box 
$ vagrant init my-box
$ vagrant up
```

* 社内のリポジトリにboxファイルをupして共有ってユースケースもあると思うが、**boxファイルはサイズがデカイのでリソースの観点でちと優しくないのが難点**

* 設定モロモロ

```
config.vm.provider "virtualbox" do |v|

# GUIモードを有効化
  v.gui = true

# VM名設定
  v.name = "my_vm"


# VBoxManageを用いたカスタマイズ
  # :id パラメータは、VBoxManageで必要な"ID" 
  v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]

  # メモリのサイズ変更はショートカットな書き方あり
  v.memory = 1024

end
```

##### VBoxManage

* virtualboxのCUI, VBoxManageコマンド。"VBoxManage [サブコマンド] [引数]" という形式で実行する
  * [引数] には UUIDか名前(vm名とか) を指定するのが主
  * UUIDとはVDIファイルに採番されるIDのこと。list hddsでVMイメージの物理ファイルを確認したら表示される

```
$ VBoxManage list
"berkstest_default_aaa" {xx-yy-xx-hoge}
```

* list : 情報表示。作成したVMの一覧表示は"list vms"で。vms以外のオプションはヘルプを参照  
  * hdds : イメージディスクの一覧
  * groups
  * systemproperties
* showvminfo : マシンの情報表示
* showhdinfo : VMイメージディスクの情報表示
* modifyvm : 停止中のVMのプロパティ変更
* snapshot : スナップショットの取得


#### VMWare


#### AWS


### provisioning

* 基本は "provision" メソッドでprovisonerを指定する形式

```
Vagrant.configure("2") do |config|
  # ... other configuration

  config.vm.provision "shell", inline: "echo hello"

  config.vm.provision "shell" do |s|
    s.inline = "echo hello"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.playbook = "provisioning/playbook.yml"
  end

  config.vm.provision "chef_solo" do |chef|
    chef.add_recipe "apache"
    chef.cookbooks_path = "my_cookbooks"
    chef.cookbooks_path = ["cookbooks", "my_cookbooks"]

    chef.roles_path = "roles"
    chef.add_role("web")

    chef.data_bags_path = "data_bags"

    chef.json = {
      "apache" => {
        "listen_address" => "0.0.0.0"
      }
    }

    chef.node_name = "foo"
  end

  config.vm.provision "chef_client" do |chef|
    chef.chef_server_url = "http://mychefserver.com:4000/"
    chef.validation_key_path = "validation.pem"

    # Add a recipe
    chef.add_recipe "apache"

    # Or maybe a role
    chef.add_role "web"
  end

  config.vm.provision "docker", images: ["ubuntu"]

  config.vm.provision "docker" do |d|
    d.pull_images "ubuntu"
    d.pull_images "vagrant"

    d.run "rabbitmq"

    d.run "ubuntu",
      cmd: "bash -l",
      args: "-v '/vagrant:/var/www'"
  end

  config.vm.provision "puppet" do |puppet|
    puppet.manifests_path = "my_manifests"
    puppet.manifest_file = "default.pp"
  end
end
```

* 主なprovisionerとオプション
  * shell : シェルスクリプト
      * inline : 実行コマンドライン
      * path : スクリプトファイルの(Vagrantファイルからの相対)パス。ゲストVMにアップロードされて実行される(gistのような挙動？)
          * リモートのファイルもURLで指定可能
      * args : 実行時引数

  * ansible : [Chefがつらい人のためのAnsibleのはなし - ゆううきブログ](http://yuuki.hatenablog.com/entry/2013/08/13/220330)
  * chef_solo : Chef Solo  __※knife solo利用時との比較イメージとしては、knife solo init で作成される定義集(nodesとかrolesとかdata_bagとか)をVagrantfileで定義する__ というイメージ
      * cookbooks_path : レシピのパス
      * data_bags_path : data_bagsのパス
      * roles_path : ロールファイルのパス
      * add_recipe(メソッド) : 実行するレシピを追加する
      * run_list : 実行するレシピ群を配列で定義する
      * add_role(メソッド) : 実行するロールを追加する
      * json : レシピ毎のattributeをJSONで定義する
      * log_level : ログレベル
      * ***plugins/provisioners/chef/config/base.rb*** を参照
  * chef_client : (既存の)Chef Serverのchef client
  * docker : Dockerのコンテナ向け
      * image : 
      * version : 
  * pappet : Puppet
  * papper_server : 
  * salt : 

===

## 各種ディレクトリ構成 とログ
VirtualBoxの分も含めて

```
$HOME
└─VirtualBox\ VMs
  └── centos65_86_vmcentdocker_1399460058722_41674
      ├── Logs
      │  ├── VBox.log  *実行時等のログ
      ├── box-disk1.vmdk
      ├── centos65_86_vmcentdocker_1399460058722_41674.vbox  * boxの定義ファイル(?)
      └── centos65_86_vmcentdocker_1399460058722_41674.vbox-prev


$VAGRANTFILE_DIR
  .vagrant
  └── machines
      └─$BOX_NAME
          └── virtualbox
              ├── action_set_name
              └── id


$HOME
└.vagrant.d─
  ├── boxes
  │   ├── centos65_86
  │   │   └── 0
  │   │       ├── 0
  │   │       │   └── virtualbox
  │   │       │       ├── Vagrantfile
  │   │       │       ├── box-disk1.vmdk
  │   │       │       ├── box.ovf  * 作成した仮想マシンの定義OVF(Open Virtualization Formatファイル)
  │   │       │       └── metadata.json
  │   │       └── virtualbox
  │   │           ├── Vagrantfile
  │   │           ├── box-disk1.vmdk
  │   │           ├── box.ovf
  │   │           └── metadata.json
  │   ├── ubuntu_1304_amd64
  │   │   └── 0
  │   │       └── virtualbox
  │   │           ├── \034%B5%D2uu%B8A%E9^%84%A5%91e�\206:%80%EB%9C&E
  │   │           │   └── a%ABW:%B0[K&%E9A\030#;=c%8A%CE!mk%F72\032%AD\005%E6\030%A1fj%D5dP\023J%99%A7%92C%B3%8B%BA6%FA\022;v1%E3,%A6Y%B9S%92xK%89x%CBum%CFS%E5Y)\fx%8F%A2Z6%ED%8A\021%B0%F7
  │   │           ├── Vagrantfile
  │   │           ├── box-disk1.vmdk
  │   │           ├── box.ovf
  │   │           └── metadata.json
  │   └── ubuntu_1304_amd64_noguest
  │       └── 0
  │           └── 0
  │               └── virtualbox
  │                   ├── Vagrantfile
  │                   ├── box-disk1.vmdk
  │                   ├── box.ovf
  │                   └── metadata.json
  ├── data
  ├── gems
  │   ├── build_info
  │   │   ├── childprocess-0.5.3.info
  │   │   ├── micromachine-1.1.0.info
  │   │   └── vagrant-vbguest-0.10.0.info
  │   ├── cache
  │   │   ├── childprocess-0.5.3.gem
  │   │   ├── micromachine-1.1.0.gem
  │   │   └── vagrant-vbguest-0.10.0.gem
  │   ├── doc
  │   ├── gems *主にpluginのgem置き場
  │   │   └── vagrant-vbguest-0.10.0
  │   │       ├── CHANGELOG.md
  │   │       ├── Gemfile
  │   │       ├── LICENSE
  │   │       ├── Rakefile
  │   │       ├── Readme.md
  │   │       ├── lib
  │   │       │   ├── vagrant-vbguest
  │   │       │   │   ├── command.rb
  │   │       │   │   ├── config.rb
  │   │       │   │   ├── core_ext
  │   │       │   │   │   └── string
  │   │       │   │   │       └── interpolate.rb
  │   │       │   │   ├── download.rb
  │   │       │   │   ├── errors.rb
  │   │       │   │   ├── hosts
  │   │       │   │   │   ├── base.rb
  │   │       │   │   │   └── virtualbox.rb
  │   │       │   │   ├── installer.rb
  │   │       │   │   ├── installers
  │   │       │   │   │   ├── base.rb
  │   │       │   │   │   ├── debian.rb
  │   │       │   │   │   ├── linux.rb
  │   │       │   │   │   ├── redhat.rb
  │   │       │   │   │   └── ubuntu.rb
  │   │       │   │   ├── machine.rb
  │   │       │   │   ├── middleware.rb
  │   │       │   │   ├── rebootable.rb
  │   │       │   │   ├── vagrant_compat
  │   │       │   │   │   ├── vagrant_1_0
  │   │       │   │   │   │   ├── command.rb
  │   │       │   │   │   │   ├── download.rb
  │   │       │   │   │   │   ├── rebootable.rb
  │   │       │   │   │   │   └── vm_compatible.rb
  │   │       │   │   │   ├── vagrant_1_1
  │   │       │   │   │   │   ├── command.rb
  │   │       │   │   │   │   ├── download.rb
  │   │       │   │   │   │   ├── rebootable.rb
  │   │       │   │   │   │   └── vm_compatible.rb
  │   │       │   │   │   ├── vagrant_1_2
  │   │       │   │   │   │   ├── command.rb
  │   │       │   │   │   │   ├── download.rb
  │   │       │   │   │   │   ├── rebootable.rb
  │   │       │   │   │   │   └── vm_compatible.rb
  │   │       │   │   │   └── vagrant_1_3
  │   │       │   │   │       ├── command.rb
  │   │       │   │   │       ├── download.rb
  │   │       │   │   │       ├── rebootable.rb
  │   │       │   │   │       └── vm_compatible.rb
  │   │       │   │   ├── vagrant_compat.rb
  │   │       │   │   └── version.rb
  │   │       │   ├── vagrant-vbguest.rb
  │   │       │   └── vagrant_init.rb
  │   │       ├── locales
  │   │       │   └── en.yml
  │   │       └── vagrant-vbguest.gemspec
  │   ├── ruby
  │   │   └── 2.0.0
  │   └── specifications
  │       ├── childprocess-0.5.3.gemspec
  │       ├── micromachine-1.1.0.gemspec
  │       └── vagrant-vbguest-0.10.0.gemspec
  ├── insecure_private_key
  ├── plugins.json
  ├── rgloader
  │   └── loader.rb
  ├── setup_version
  └── tmp
      ├── 1391584190-install.sh
      └── vagrant-box-add-temp-20140507-60789-sr1dvo
          └── \034%B5%D2uu%B8A%E9^%84%A5%91e�\206:%80%EB%9C&E
              └── a%ABW:%B0[K&%E9A\030#;=c%8A%CE!mk%F72\032%AD\005%E6\030%A1fj%D5dP\023J%99%A7%92C%B3%8B%BA6%FA\022;v1%E3,%A6Y%B9S%92xK%89x%CBum%CFS%E5Y)\fx%8F%A2Z6%ED%8A\021%B0%F7 [error opening dir]

```




===

## ライブラリ・プラグイン

### インストール/アンインストール

```
$ vagrant plugin install [plugin name]

$ vagrant plugin uninstall [plugin name]

$ vagrant plugin list  # インストール済プラグインの確認
```

* 代表的なプラグイン
  * vagrant-berkshelf
  * vagrant-omnibus
  * vagrant-vbguest
  * sahara

#### vagrant-vbguest プラグイン
* vagrant up時にちょくちょく見かけるバージョン差異エラー`Guest Additions Version: 4.1.18``VirtualBox Version: 4.3` の解消はコレで
    * インストールして`vagrant vbguest`する

```
% vagrant plugin install vagrant-vbguest

% vagrant up
% vagrant vbguest
% vagrant vbguest --status
```


### Saharaの導入
"Sahara というサードパーティライブラリは Vagrant に commit & rollback ができる機能を追加します"

```
$ vagrant gem install sahara
$ (rbenv rehash)
```

* sandbox on でsandboxモード開始

```
$ vagrant up
$ vagrant sandbox on
[default] - Enabling sandbox
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
```

* guest側で適当にオペレーション

```
[vagrant@vmcheftry ~]$ vi aho.txt

[vagrant@vmcheftry ~]$ ls -l
合計 8
-rw-rw-r-- 1 vagrant vagrant    5  3月 13 10:59 2013 aho.txt
-rw-r--r-- 1 vagrant vagrant 1506  7月 10 22:14 2012 postinstall.sh
```

* 巻き戻す

```
$ vagrant sandbox rollback  # ★再起動される
[default] - powering off machine
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
[default] - roll back machine
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
[default] - starting the machine again

$ vagrant sandbox commit
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
```

* 確認

```
$ vagrant ssh

[vagrant@vmcheftry ~]$ ls -l # ★rollback前に作成したファイルが消えている
合計 4
-rw-r--r-- 1 vagrant vagrant 1506  7月 10 22:14 2012 postinstall.sh
```

* 自作はこちらを参照 [Plugin Development Basics - Plugins - Vagrant Documentation](http://docs.vagrantup.com/v2/plugins/development-basics.html)

===

## ブクマ
* [ShawnMcCool/vagrant-chef](https://github.com/ShawnMcCool/vagrant-chef)

## Ver 1.5 CHANGELOG
[Vagrant 1.5 and Vagrant Cloud - Vagrant](http://www.vagrantup.com/blog/vagrant-1-5-and-vagrant-cloud.html)

* Vagrant Cloud - Vagrant環境(box)の公開・共有クラウドサービス
    * 現在開発中の機能 - APIアクセス、boxの使用ログ(証跡、統計)、アクセスコントロール、カスタムドメイン、etc 
* Vagrant Share - Vagrant Cloudへ公開したVagrant環境を共有するツール
    * 主な特徴
        * HTTPアクセス - 共有したVagrant環境にApacheなりNginxなりでWebサーバを立てている場合、独自ドメインが割り当てられてHTTPアクセスが可能になる
        * SSHアクセス - 共有したVagrant環境にSSHログインが可能になる
        * ローカルPC上に同一構成のVMを __静的IPを割り当てて__ 構築
    * `vagrant login`コマンド - Vagrant Cloudへ、作成したアカウントでログイン(shareとconnectはログインしたら実行可能になる)
    * `vagrant share`コマンド - 実行中のVagrant環境を共有する。デフォルトではHTTPアクセス(権)が共有される
        * `vagrant share --ssh` - SSHアクセス(権)を共有する
            * key-pairのパスワードを設定、接続する側はそのパスワードでログイン
            * public keyは共有するVagrant環境へ、private keyはz共有環境を管理しているサーバ(Vagrant Cloud?)へ設置される
        * `Ctrl+C` でshareを終了 
    * `vagrant connect SHARE-NAME`コマンド - 共有中のVagrant環境に接続し、同構成のVMをローカルに構築する（ように見せる）。その際、静的IPが割り当てられる
        * 公式のデモ動画にもあるが、shareでは __サーバーの状態も共有している__ ので、例えばredisであるkeyでset valueした場合、connectした側でも同じkey-valueが見える状態になっている
        * `vagrant connect --ssh` - SSHアクセス有りで共有中のVagrant環境にSSHログインする
    * 当然セキュリティ上の留意点も存在しており、対策が必要
        * share時のオプションでセキュリティ強化
            * `--disable-http` - HTTPアクセスを使用不可に
            * `--ssh-once` - SSHアクセスを1度だけ許可する
        * vagrant share は、end-to-endのTLS接続だが、 __暗号化されていないTCPストリームが流れている箇所も一部存在する__
        * vagrant share した共有状態は、 __1時間で自動的に終了__ する
    * 通信イメージ。`Connect proxy VM`は、vagrant connect を実行した側のマシンで立ち上がるproxy専用の小さいVM(使用RAM=13MB程度のサイズのもの)

    ```
                                 +--------------------------------------+
                                 |        Remote server                 |
                                 |--------------------------------------|
                                 |                                      |
                                 |          +----------------+          |
                                 |          | *Remote proxy  |          |
                                 |          |                |          |
                                 |          +----------------+          |
                                 |              ^      ^                |
                                 +--------------|------|----------------+
                                                |      |
         Internet                               |   (SOCKS5 Protocol)
                                                |      |
     +------------------------------------------|---+--|--------------------------------------------+
                                                |   |  |
                                                |   |  |
                +----------------------------+  |   |  |  +------------------------------+
                |   share src machine        |  |   |  |  |    share dst machine         |
                |----------------------------|  |   |  |  |------------------------------|
                |                            |  |   |  |  |                              |
                |                            |  |   |  |  |                              |
                |     +----------------+     |  |   |  |  |    +-------------------+     |
                |     |*Local proxy    +--------+   |  +------>|*Connect proxy VM  |     |
                |     +----------------+     |      |     |    +-------------------+     |
                |                            |      |     |                              |
                |                            |      |     |                              |
                +----------------------------+      |     +------------------------------+
                                                    |
                                                    +
    ```

* Boxes 2.0。box名の省略形(`[user_name]/[box_name]`)のサポート
* Rsyncによるホスト<=>ゲスト間のフォルダ同期。 __速い__
    * Vagrantfileで設定 - `config.vm.synced_folder ".", "/vagrant", type: "rsync"`して、upやreload時に同期を実行
    * `vagrant rsync`コマンドで同期だけ実行
    * `vagrant rsync-auto`で、変更検知→同期の自動実行をバックグラウンドで実行し続ける
* Windows対応。SMBによるフォルダ同期、Hyper-Vサポート
* パスワード・ベースのSSH認証
* `vagrant plugin update`コマンドによるプラグインの更新
* サポートするLinuxディストリビューションの追加。

### トラブルシューティング

* vagrant hogeすると`Malformed version number string virtualbox`が発生する
    * [can't start my VM on Vagrant 1.5.1 · Issue #3195 · mitchellh/vagrant](https://github.com/mitchellh/vagrant/issues/3195)
    * とりあえず `echo "1.1" > ~/.vagrant.d/setup_version` してみた（実行前は"1.5"） => __OK__


## ver 1.6

### アップデート後のハマりどころ
* 1.5同様、vagrant hogeすると`Malformed version number string virtualbox`が発生する
    * ~/.vagrant.d/setup_version を1.6にしたら、`Vagrant failed to initialize at a very early stage`が発生
    * `vagrant init`からやり直す => 同様のエラーが発生
        * `~/.vagrant.d`を消す => (3階層目ぐらいからrwの権限すら無いのでchmodして消す) => __OK__
    * `vagrant up` =>  __OK__

### Dockerを "provisionerとして" 使用する
* `pull_images`すればimageのダウンロードと同時にdockerのインストールも行われる

### Dockerを "providerとして" 使用する
* DockerコンテナをVMの __providerとして__ 指定することが可能
* `vagrant_vagrantfile`オプション - コンテナの設定を記述するVagrantfile内で使用, __コンテナを動かす為(だけ)のVM設定を別途記載したVagrantfileを用意し、そのパスを指定__ する
    * 例

    ```ruby
    config.vm.define "elasticsearch" do |v|
      v.vm.provider "docker" do |d|
        d.image = "dockerfile/elasticsearch"
        d.ports = ["9200:9200"]
        d.vagrant_vagrantfile = "./Vagrantfile.proxy"  # **コンテナ稼働用VM設定ファイル
      end
    end
    ```
