# Dockerについて調べたのでヅラヅラとまとめ
(last updated: 2014/08)

## 概念とか用語とか、とりあえず押さえとくべきもの

* `lxc`
    * [LXC - Wikipedia](http://ja.wikipedia.org/wiki/LXC)
    * __OSレベル__ の仮想化ソフトウェア
        * 完全仮想化（ハードウェアをエミュレート、オーバーヘッド大） - KVM etc
        * 準仮想化（ゲストOSがホストOSのAPIを使用、オーバーヘッド中） - Xen, Hyper-V etc。Virtualboxもこちらか
    * 仮想機械ではなく、個別のプロセスとネットワークスペースを作り出す仮想環境
    * ホストマシンの単一のカーネルの上で動作する。他の仮想化ソフトウェア（例えばXenとか）はVM単位で複数のカーネルが動く
    * Linuxカーネル 2.6.29から利用可能
    * cgroupsに依存（cgoups - プロセスグループのリソース(CPU、メモリ、ディスクI/Oなど)の利用を制限・隔離するLinuxカーネルの機能）
    * 完全仮想化や準仮想化と比較して
        * 実行速度が高速
        * セキュリティ面で優位
        * ハードウェアリソースの有効活用面で優位
* `image`（イメージ） - 
* `container`（コンテナ） - 

## 考察

* [最近のサーバの抽象化について - As a Futurist...](http://blog.riywo.com/2013/08/06/174100) 
    * 物理サーバの場合、「OS が プロセス へ リソース を割り振る」 = リソースコントロールやりづらい。ファイルシステム, CPU, RAM...
    * 異なるOSを同居して使うことが出来ない（使いたいライブラリがDebian版しかない, etc）
    * AWSのようなクラウドサービスは、 __欲しいサーバーを欲しい時に素早く入手できる__ 
        * ただしパフォーマンス（特にI/Oの）とのトレードオフ
    * Dockerは、 __プロセスをあたかもLinuxのコンテナの様にしてしまう__
        * 軽量
        * 柔軟なリソース割り振り
            * __じゃあ "どうやって" 割り振るか__ は今後の課題
            * MesosとかYARNとか
        * __ホストとゲストでディストリビューションやそのバージョンが違ってても動く__
        * 他の仮想化と比較してオーバーヘッドもない（少ない）
    * Dockerは __イメージの使い回しが可能__
        * 自社の物理サーバーで稼働させていたコンテナイメージをAmazon EC2上に移す etcが容易 

## Install

### CentOS

```
$ sudo yum -y install docker-io
$ sudo /etc/init.d/docker start
```

#### debugを有効にしてサービス起動





## イメージの公開/共有
[Share Images via Repositories - Docker Documentation](http://docs.docker.io/en/latest/use/workingwithrepository/)

> Docker is not only a tool for creating and managing your own containers – Docker is also a tool for sharing.

* central index [Home | Docker Index](https://index.docker.io/) で様々なイメージが公開されている
* central index上のイメージは `docker search` で検索可能
* 自作イメージを公開したければ、まず `docker login` でcentral indexにアカウントを登録
* イメージ作成/push時には、`登録したアカウント名/イメージ名` を使用するのが慣例

```
$ sudo docker commit CONTAINER [myname]/[image_name]
$ sudo docker push [myname]/[image_name]
```

* githubのアカウントと連携することで、 __Trusted Builds__ が使用可能
    * githubにupしたDockerfileを含んだプロジェクト をdocker.ioが自動でビルドしてくれるサービス
    * 結果は [Builds | Docker Index](https://index.docker.io/builds/) にフィードバックされる
* プライベートリポジトリも使用可能
    * プレイベートリポジトリサーバーは [dotcloud/docker-registry](https://github.com/dotcloud/docker-registry) を使用して構築する
    * central indexを含んだ複数のリポジトリを使用する場合、`$HOME/.dockercfg`ファイルにそれぞれの認証情報をJSON形式で記述

    ```
    {
         "https://index.docker.io/v1/": {  // central indexのキーは "https://index.docker.io/v1/" 固定
                 "auth": "xXxXxXxXxXx=", // authには "[username]:[password]" をbase64エンコードした文字列を設定
                 "email": "email@example.com"
         },
         "https://my-registry.com": {
                 "auth": "XxXxXxXxXxX=",
                 "email": "email@my-registry.com"
         }
    } 
    ```

## ポートフォワード
[Redirect Ports - Docker Documentation](http://docs.docker.io/en/latest/use/port_redirection/)

* コンテナに割り振られるIPアドレスは、 __ホストマシン内でローカルなもの__
    * `docker inspect` の出力で確認可能
* コンテナ（のポート）はホストマシン外部からはunreachable
* コンテナのIPアドレスが起動の度に変わるので「コンテナAからコンテナBのポートに接続したい」なんてケースも起動の度に設定をいじらないと駄目
* つまり単にdocker runしただけのコンテナでは、他コンテナやホストマシンetcとの相互通信が不可能/不便である
    * __ポートバインディング（like マッピング, フォワーディング）を使って解決__
    * 非ローカル（＝ホストマシンやホストマシン内のコンテナ 以外）からコンテナのサービスに接続したい場合も同様
* docker run時に`-P`オプションを使用した場合、ホスト側でバインドするポートは`49000..49900`の範囲で未使用のものが自動選択される
    * vagrantを使用している場合、Vagrantfileの`:forwarded_port`属性でこれらのポートフォワードを指定しておけば、ローカルマシン => VM => dockerコンテナ 間のポート通信（フォワード）を実現可能
* もちろん明示的に指定することも可能で、docker runする際に`-p`オプションを使用すると、ホストマシン<=>コンテナ間の指定ポート同士がバインディングされる
* 以上から、 __dockerコンテナではホストとのポートバインディングだけ気にしておけばよくiptablesのようなアクセス制御は行わない・アクセス制御はホストマシン"だけ"で行う__ のがあるべき姿ｎのかなと

## ネットワーク設定
[Configure Networking - Docker Documentation](http://docs.docker.io/en/latest/use/networking/)

* コンテナへの接続はブリッジ接続
* デフォルトでは `docker0` というブリッジを使う
* dockerのデーモン起動時に、未使用のIP範囲を検索し、docker0ブリッジに割り当てられる
* コンテナ実行時にdocker0ブリッジと紐付いた仮想的なネットワークインタフェースが割り当てられる
    * 同時にdocker0に割り当てられたIP範囲のIPアドレスが割り当てられる
* docker0のIPアドレスが、コンテナから見たgatewayアドレスとなる
* docker0はXenのxenbr0と似てるようで違う
    * xenbr0は物理ネットワーク上に存在させる（ように見せる）為の仮想ブリッジ、docker0はそのような見え方はさせない
        * ポートフォワードの項で書いたとおり外からはunreachable（ポートフォワードを使え というポリシー）
        * コンテナ同士でover ブリッジな通信を可能にしたい場合は、`-icc=true` オプションを指定する（デフォルトはfalse）
* docker0はIPv6が有効になってる模様
* コンテナを実行するとdocker0のブリッジ情報のインタフェース欄に`vethXXXX`というID？が表示される
    * __ホストのvethXXXX と コンテナのeth0__ が接続ペア

```
[vagrant@vmdocker1 ~]$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.fe239cee3696       no              

[vagrant@vmdocker1 ~]$ ifconfig docker0
docker0   Link encap:Ethernet  HWaddr FE:23:9C:EE:36:96
          inet addr:172.17.42.1  Bc
          ast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::c8e2:13ff:fee7:7766/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:2940 errors:0 dropped:0 overruns:0 frame:0
          TX packets:12169 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:120698 (117.8 KiB)  TX bytes:14877602 (14.1 MiB)

[vagrant@vmdocker1 ~]$ docker run -t -i -d hoge /bin/bash

[vagrant@vmdocker1 ~]$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.fe239cee3696       no              vethowR0mm  # <= コンテナ実行中は仮想的なインタフェースが割り当てられる
```

* IPアドレス（のレンジ）を変更したりしたい場合
    * docker用のブリッジを追加作成
    * run時にそのブリッジを指定

```
# Stop Docker
$ sudo service docker stop

# Clean docker0 bridge and
# add your very own bridge0
$ sudo ifconfig docker0 down
$ sudo brctl addbr bridge0
$ sudo ifconfig bridge0 192.168.227.1 netmask 255.255.255.0

# Edit your Docker startup file
$ echo "DOCKER_OPTS=\"-b=bridge0\"" >> /etc/default/docker

# Start Docker
$ sudo service docker start

# Ensure bridge0 IP is not changed by Docker
$ sudo ifconfig bridge0
bridge0   Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx
          inet addr:192.168.227.1  Bcast:192.168.227.255  Mask:255.255.255.0

# Run a container
$ docker run -i -t base /bin/bash

# Container IP in the 192.168.227/24 range
root@261c272cd7d5:/# ifconfig eth0
eth0      Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx
          inet addr:192.168.227.5  Bcast:192.168.227.255  Mask:255.255.255.0

# bridge0 IP as the default gateway
root@261c272cd7d5:/# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.227.1   0.0.0.0         UG    0      0        0 eth0
192.168.227.0   0.0.0.0         255.255.255.0   U     0      0        0 eth0

# hits CTRL+P then CTRL+Q to detach

# Display bridge info
$ sudo brctl show
bridge      name    bridge id               STP enabled     interfaces
bridge0             8000.fe7c2e0faebd       no              vethAQI2QT
```

* その他、コンテナ同士の複雑めのネットワーク設定を行いたい場合は、lxc用のSDNツール [jpetazzo/pipework](https://github.com/jpetazzo/pipework) を使うとよろし

## プロセスマネージャにコンテナ自動起動を担当させる
[Automatically Start Containers - Docker Documentation](http://docs.docker.io/en/latest/use/host_integration/)

* upstart, systemd, supervisor といったプロセスマネージャでdockerコンテナ（の起動etc）を管理可能
* `-r=false` オプションを指定して、ホストサーバ再起動時のコンテナ自動起動を抑止しておく

## ディレクトリ共有

* ホストマシンと1つ以上のコンテナの間でディレクトリを共有する __data volume__
* __Union File System__ なる仕組みを使用
    * 対象data volumeはコンテナ間で共有・再利用可能
        * DBのバックアップやレプリケーションなんかに使うと良いよ。って話だけど、I/Oのパフォーマンス的にどうなんだろう・・・
    * 対象data volumeはコピーオンライトのオーバーヘッドを発生させること無く直接変更可能
    * __対象data volumeへの変更は次回のコミットには含まれない__
* ★別ホストマシン - (NFSとか) - 自ホストマシン - (data volume) - コンテナ、という3者間での共有も可能だろうか？
    * これが可能なら「例えばホストマシンを跨いだコンテナ間でのMySQLレプリケーション」とかも出来たりする？

```
# コンテナ名=DATA のコンテナで共有ディレクトリを2つ作成 ※該当コンテナはディレクトリのマウントだけ実行してすぐ終了させる
[vagrant@vmdocker1 ~]$ docker run -v /var/volume1 -v /var/volume2 --name DATA a2ca5119cdcf true

# 作成した共有ディレクトリを別コンテナ1から参照
[vagrant@vmdocker1 ~]$ docker run -t -i -rm --volumes-from DATA --name client1 a2ca5119cdcf /bin/bash

bash-4.1# ls -l /var/ | grep volume
drwx------ 2 root root 4096 Mar  5 00:35 volume1
drwx------ 2 root root 4096 Mar  5 00:35 volume2

# さらに別のコンテナ2からコンテナ1の共有ディレクトリを参照
[vagrant@vmdocker1 ~]$ docker run -t -i -rm --volumes-from client1 --name client2 a2ca5119cdcf /bin/bash

bash-4.1# ls -l /var | grep volume
drwx------ 2 root root 4096 Mar  5 00:35 volume1
drwx------ 2 root root 4096 Mar  5 00:35 volume2
```

* ★ホストマシン=>コンテナのディレクトリ共有を行って、そのコンテナから別コンテナへも共有した場合、競合は起こらない？

```
[vagrant@vmdocker1 ~]$ sudo mkdir /container
[vagrant@vmdocker1 ~]$ sudo touch /container/from_host

[vagrant@vmdocker1 ~]$ docker run -v /container:/container --name DATA a2ca5119cdcf true

[vagrant@vmdocker1 ~]$ docker run -t -i --rm --volumes-from DATA --name client1 a2ca5119cdcf /bin/bash
bash-4.1# ls -l /container/
total 0
-rw-r--r-- 1 root root 0 Mar  5 00:55 from_host
bash-4.1# touch /container/from_client1
[vagrant@vmdocker1 ~]$ ls -l /container/
total 0
-rw-r--r-- 1 root root 0 Mar  5 05:56 from_client1
-rw-r--r-- 1 root root 0 Mar  5 05:55 from_host

[vagrant@vmdocker1 ~]$ docker run -t -i --rm --volumes-from client1 --name client2 a2ca5119cdcf /bin/bash
bash-4.1# ls -l /container/
total 0
-rw-r--r-- 1 root root 0 Mar  5 00:56 from_client1
-rw-r--r-- 1 root root 0 Mar  5 00:55 from_host
bash-4.1# touch /container/from_client2
[vagrant@vmdocker1 ~]$ ls -l /container/
total 0
-rw-r--r-- 1 root root 0 Mar  5 05:56 from_client1
-rw-r--r-- 1 root root 0 Mar  5 06:09 from_client2
-rw-r--r-- 1 root root 0 Mar  5 05:55 from_host
```

## 複数コンテナのリンク
[Link Containers - Docker Documentation](http://docs.docker.io/en/latest/use/working_with_links_names/)

* --name名前付けしたコンテナ同士は "リンク" 機能で __親子関係__ を構築することが出来る
* 子コンテナ実行時に `--link 親name:alias` 形式でオプション指定
* dockerデーモン起動時の`-icc=false`設定が効いてて、デフォルトではコンテナ間の通信は許可されない。（このおかげでコンテナがセキュアに保たれてる）
* __リンクを設定するとコンテナ同士で下記にアクセス可能になる__
    * 環境変数
    * 使用しているポート
        * 親側でポート8080のみ公開している場合、子側も同様に8080のみがアクセス可能になる。他のポートをコンテナ間通信で使用することは出来ない
        * 例えばredis(ポート6379)のサーバとクライアントのコンテナを実行する場合、まず親（サーバ）を--name付きで実行した後、子（クライアント）を __-pによるポートバインディングを使わず--link付きで実行__ する
        * こうすれば __redisで必要な通信だけが許可された状態__ を作ることが出来る

* ★link機能を使ったロードバランシング(proxyとか)を試す
* ★↑で例えば1コンテナが死亡したときにもう片方にどんな影響が出るか調べる

## Ambassador
[Link via an Ambassador Container - Docker Documentation](http://docs.docker.io/en/latest/use/ambassador_pattern_linking/)

* __複数ホスト間にまたがるコンテナ同士の通信を手助けする__ 仕組み
    * バランサー/proxy のようなもんか
* `docker link`によるリンク機能は単一のホストマシン内では良い感じに機能するが、例えばコンテナで稼働させるアプリケーションを複数ホストでスケールさせたい場合には不便(というか出来ない？)。ここでAmbassadorの出番
* 例えば`(client) -> (redis)`のような接続において、redisのサーバー切り替えがあるとclientの設定書き換え __と再起動__ が必要になるかもしれない。AmbassadorでproxyしておけばAmboassadorの設定変更(+再起動)で済み、clientは稼働したままにしておける
    * redisのように接続先であるサーバーの変更/移行が容易になる

```
# MySQLで複数ホスト/複数コンテナのイメージ

         +--------------------------+        +---------------------------+
         |Server Host               |        |Client Host                |
         |                          |        |                           |
         |   +-----------------+    |        |   +------------------+    |
         |   | container sh1   |    |        |   | container ch1    |    |
         |   |-----------------|    |        |   |------------------|    |
         |   | MySQL           |    |        |   | (want to conn)   |    |
         |   +--------^--------+    |        |   +--------+---------+    |
         |            |             |        |            | 3306         |
         |   +--------+--------+    |        |   +--------v---------+    |
         |   | container sh2   |    |        |   | container ch2    |    |
         |   |-----------------|    |        |   |------------------|    |
         |   | ambassador      <-----------------+ ambassador       |    |
         |   +-----------------+    |3306    |   +------------------+    |
         +--------------------------+        +---------------------------+
```
```
Server Host $ docker run -d --name sh1 mysql_image
Server Host $ docker run -d -link sh1:sh1 -p 3306:3306 --name sh2 mysql_ambassador_image

Client Host $ docker run -d --expose 3306 -e MYSQL_PORT_3306_TCP=tcp://[Server Host IP]:3306 --name ch2 mysql_ambassador_image
Client Host $ docker run -t -i -rm -link ch2:ch2 --name ch1 app_image
```

### 周辺ツール
[Deploying Multi-Server Docker Apps with Ambassadors | CenturyLink Labs](http://www.centurylinklabs.com/deploying-multi-server-docker-apps-with-ambassadors/)

> Ambassadors are containers who’s sole purpose is to help connect containers across multiple hosts.


* [Fig | Fast, isolated development environments using Docker](http://orchardup.github.io/fig/)
    * コンテナ間(ホスト間)の通信設定を`fig.yml`という名前のyamlに記述し、`fig up`コマンドでコンテナ実行

    ```yaml
    web:
      build: .
      command: python app.py
      ports:
       - "5000:5000"
      volumes:
       - .:/code
      links:
       - redis
    redis:
      image: orchardup/redis""
    ```
    ```
    $ fig up
    ```
    ```
    $ fig stop
    ```


## with Virtualbox and Vagrant

### Vagrantfile設定
ver1.4からprovisioningにdockerサポート追加 [Docker - Provisioning - Vagrant Documentation](http://docs.vagrantup.com/v2/provisioning/docker.html)

* Dockerfileをbuildしたい

```
# Dockerfileがあるディレクトリを指定して build_image する
config.vm.provision "docker" do |d|
    d.build_image "/vagrant/app"
end
```

* imageをpullしたい

```
# centosのimageをpull
config.vm.provision "docker" do |d|
    d.pull_images "centos"
end
```

* vagrant up時にdocker runしたい

```
# rabbitmqのimageを使用してコンテナを作成したい
config.vm.provision "docker" do |d|
    d.run "rabbitmq"
end

# ubuntuのimageを使用してコンテナを作成し、コマンドを実行したい
config.vm.provision "docker" do |d|
    d.run "ubuntu",
    cmd: "bash -l",
    args: "-v '/vagrant:/var/www'"  # vagrantの共有ディレクトリ(＝ローカルマシンでvagrant upを実行したディレクトリ) をコンテナの/var/wwwにマウント = 
end

# 同じイメージを使用して複数コンテナを作成したい
config.vm.provision "docker" do |d|
    d.run "db-1", image: "user/mysql"
    d.run "db-2", image: "user/mysql"
end
```

### vm起動・確認

```
% vagrant up
% vagrant ssh

$ which docker
/usr/bin/docker

$ sudo service docker status
docker (pid  3244) is running...

$ docker -v
Docker version 0.8.0, build cc3a8c8/0.8.0

$ docker info
Containers: 1
Images: 1
Driver: devicemapper
 Pool Name: docker-8:1-131916-pool
 Data file: /var/lib/docker/devicemapper/devicemapper/data
 Metadata file: /var/lib/docker/devicemapper/devicemapper/metadata
 Data Space Used: 631.6 Mb
 Data Space Total: 102400.0 Mb
 Metadata Space Used: 0.9 Mb
 Metadata Space Total: 2048.0 Mb

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
centos              6.4                 539c0211cd76        11 months ago       300.6 MB
centos              latest              539c0211cd76        11 months ago       300.6 MB
```

### docker runして何かしらコマンド実行・確認

* rootで実行
* hostnameはハッシュ
* IPアドレスは172...

```
$ sudo docker run -t centos /bin/bash -c "hostname && ps aux && ifconfig"
da65688cbf66
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  1.0  0.1  11296  1304 ?        S    00:03   0:00 /bin/bash -c hostname && ps aux && ifconfig
root         7  0.0  0.1  13360  1048 ?        R    00:03   0:00 ps aux
eth0      Link encap:Ethernet  HWaddr 86:73:E6:8A:84:51
          inet addr:172.17.0.2  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::8473:e6ff:fe8a:8451/64 Scope:Link
          UP BROADCAST  MTU:1500  Metric:1
          RX packets:1 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:90 (90.0 b)  TX bytes:90 (90.0 b)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)


```

### シェルにログインしてみる
`-i` オプション

```
$ sudo docker run -i -t centos /bin/bash
bash-4.1# 
```

### ポート待ち受けプログラムも起動させてみる
ホストVMとdockerコンテナ双方にnetcatがインストールされていること

```
$ PORT=10080
$ CONTAINER=$(sudo docker run -d -p ${PORT} centos /usr/bin/nc -l -p ${PORT})  # デタッチ（バックグラウンド）実行
$ PUBLIC_PORT=$(docker port ${CONTAINER} ${PORT} | awk -F":" '{print $2}')  # publicポート確認
$ echo "Hello World!" | nc 127.0.0.1 ${PUBLIC_PORT}
```

### 特記事項

* `vagrant halt`で停止後、同内容のVagrantfileで再度`vagrant up`した際には、実行したdockerコンテナ/作成したdockerイメージは(当然)消える


## Dockerfile
[Dockerfile Reference - Docker Documentation](http://docs.docker.io/en/latest/reference/builder/), [Dockerfile Best Practices](http://crosbymichael.com/dockerfile-best-practices.html)

* 真っさらではなく必要なミドル/ソフトがインストールされたコンテナを作成したい
    * run -i して自らインストールや設定を行い、それらが済んだ状態のコンテナをcommitして使用する
    * __Dockerfileを使う__
* 基本的な使い方
    * __Dockerfileにコンテナ起動時の処理等を記述、その出力先ディレクトリ上で `docker build` を実行すると結果が新規イメージへcommitされる__
    * 作成されたイメージでコンテナを実行する
* 書式
    * 基本は`INSTRUCTION(命令) arguments(引数)`の形。命令は大文字で書きましょう、強制ではないが引数との区別をしやすくする為
    * Dockerfileは上から順番に評価/処理される
    * __最初の命令は`FROM`でないと駄目__
    * コメントは`# Comment` シャープを使う
    * 1つのDockerfileのビルドで複数イメージを作成可能
        * __FROM__ で始まるコンテナ定義を複数書けば良い
* 命令
    * `FROM image` - ベースとなるイメージファイルを指定
    * `MAINTAINER name` - メンテナ名
    * `EXPOSE port` - 公開するポートを指定
    * `ENV key value` - 環境変数のkey, valueを指定
        * `docker run -e KEY=VALUE` でコンテナ実行時にも指定可能。 __ENVで定義済のKEYを指定した場合は上書きされる = 実行時引数 的な役割__
    * `ADD src dest` - srcからdestへのファイルコピー。srcはホストマシン __かリモートURL__ 、destはコンテナ
    * `RUN command` `RUN ["executable", "param1", "param2"]` - 現在指定中（ビルド中）のイメージでコマンドを実行し、結果をコミットする。Dockerfile内	の後続処理でそのイメージを使用する

    ```
    RUN /usr/bin/wc -l
    RUN ["/usr/bin/wc","-l"]
    ```

    * `CMD ["executable","param1","param2"]` `CMD ["param1","param2"]` `CMD command param1 param2` - コンテナ起動時に実行するコマンドを指定する。 __RUNと違って結果をコミットしたりはしない__
        * 複数書いても、実際に動くのは最後のCMDのみ
        * __CMDの目的は、コンテナ実行時のデフォルトの状態（や挙動）を与える__ こと。コンテナ実行直後にsshdを立ち上げておきたい とか
        * 挙動的には `docker commit -run '{"Cmd": command}'` と同じ

    * `ENTRYPOINT ["executable", "param1", "param2"]`, `ENTRYPOINT command param1 param2` - コンテナ起動時に実行するコマンドを指定する。 __CMDと同様結果をコミットしたりはしない__
        * 複数書いても、有効になるのは最後のENTRYPOINTのみ
    * ___CMDとENTRYPOINTの違いと、使用例___
        * CMD, ENTRYPOINTで __オプションなしのコマンド__ を記述し、`docker run CONTAINER_ID <command>`した場合の挙動
            * 参考 : [How to use entrypoint in Docker Builder - Servers as Software](http://www.kstaken.com/blog/2013/07/06/how-to-use-entrypoint-in-a-dockerfile/)
            * CMD 使用時

            ```
            # exec non-arg command using CMD
            FROM centos
            MAINTAINER hoge

            CMD ls
            ```
            ```
            # コマンド引数指定なし
            % docker run --rm IMAGE_ID
            bin
            boot
            dev
            etc
            home
            :
            ```
            ```
            # コマンド引数指定あり -> 指定したコマンドが実行される, CMDで指定したコマンドは無視(公式document的に言うと上書き)される
            % docker run --rm IMAGE_ID ls -l
            dr-xr-xr-x  2 root root  4096 Apr  1  2013 bin
            dr-xr-xr-x  2 root root  4096 Sep 23  2011 boot
            drwxr-xr-x  5 root root  4096 May  2 01:57 dev
            drwxr-xr-x 43 root root  4096 May  2 01:57 etc
            drwxr-xr-x  2 root root  4096 Sep 23  2011 home
            :
            ```

            * ENTRYPOINT 使用時

            ```
            # exec non-arg command using ENTRYPOINT
            FROM centos
            MAINTAINER hoge

            ENTRYPOINT ls
            ```
            ```
            # コマンド引数指定なし
            % docker run --rm IMAGE_ID
            bin
            boot
            dev
            etc
            home
            :
            ```
            ```
            # コマンド引数指定あり -> ENTRYPOINTのコマンドがそのまま実行される, コマンド引数が無視される
            % docker run --rm IMAGE_ID ls -l
            bin
            boot
            dev
            etc
            home
            :
            ```

        * CMD, ENTRYPOINTで __引数ありのコマンド__ を記述し、`docker run CONTAINER_ID <command>`した場合の挙動
            * CMD 使用時

            ```
            # exec with-arg command using CMD
            from centos
            maintainer hoge

            CMD ls -l
            ```
            ```
            # コマンド引数指定なし
            % docker run --rm IMAGE_ID
            dr-xr-xr-x  2 root root  4096 Apr  1  2013 bin
            dr-xr-xr-x  2 root root  4096 Sep 23  2011 boot
            drwxr-xr-x  5 root root  4096 May  2 01:57 dev
            drwxr-xr-x 43 root root  4096 May  2 01:57 etc
            drwxr-xr-x  2 root root  4096 Sep 23  2011 home
            :
            ```
            ```
            # コマンド引数指定あり -> 指定したコマンド(とオプション)が実行される, CMDで指定したコマンドは無視(公式document的に言うと上書き)される
            % docker run --rm IMAGE_ID ls -lh
            dr-xr-xr-x  2 root root 4.0K Apr 12 01:10 bin
            drwxr-xr-x  4 root root  80K May  2 06:30 dev
            drwxr-xr-x 40 root root 4.0K May  2 06:30 etc
            drwxr-xr-x  2 root root 4.0K Sep 23  2011 home
            dr-xr-xr-x  2 root root  4096 Apr  1  2013 bin
            dr-xr-xr-x  2 root root  4096 Sep 23  2011 boot
            drwxr-xr-x  5 root root  4096 May  2 01:57 dev
            drwxr-xr-x 43 root root  4096 May  2 01:57 etc
            drwxr-xr-x  2 root root  4096 Sep 23  2011 home
            :
            ```

            * ENTRYPOINT 使用時

            ```
            # exec with-arg command using ENTRYPOINT
            FROM centos
            MAINTAINER hoge

            ENTRYPOINT ls -l
            ```
            ```
            # コマンド引数指定なし
            % docker run --rm IMAGE_ID
            dr-xr-xr-x  2 root root  4096 Apr 12 01:10 bin
            drwxr-xr-x  4 root root 81920 May  2 06:37 dev
            drwxr-xr-x 40 root root  4096 May  2 06:37 etc
            drwxr-xr-x  2 root root  4096 Sep 23  2011 home
            :
            ```
            ```
            # コマンド引数指定あり -> ENTRYPOINTのコマンドがそのまま実行される, コマンド引数が無視される
            % docker run --rm IMAGE_ID ls -lh
            dr-xr-xr-x  2 root root  4096 Apr 12 01:10 bin
            drwxr-xr-x  4 root root 81920 May  2 06:37 dev
            drwxr-xr-x 40 root root  4096 May  2 06:37 etc
            drwxr-xr-x  2 root root  4096 Sep 23  2011 home
            :
            ```

        * CMD, ENTRYPOINTで __`["command", "paramN"]`形式による引数付きコマンド__ を記述し、`docker run CONTAINER_ID <command>`した場合の挙動
            * CMD 使用時

            ```
            # exec params command using CMD
            from centos
            maintainer hoge

            CMD ["ls", "-l"]
            ```
            ```
            # コマンド引数指定なし
            % docker run --rm IMAGE_ID
            dr-xr-xr-x  2 root root  4096 Apr  1  2013 bin
            dr-xr-xr-x  2 root root  4096 Sep 23  2011 boot
            drwxr-xr-x  5 root root  4096 May  2 01:57 dev
            drwxr-xr-x 43 root root  4096 May  2 01:57 etc
            drwxr-xr-x  2 root root  4096 Sep 23  2011 home
            :
            ```
            ```
            # コマンド引数指定あり -> 指定したコマンド(と引数)が実行される, CMDで指定したコマンドは無視(公式document的に言うと上書き)される
            % docker run --rm IMAGE_ID ls -lh
            dr-xr-xr-x  2 root root 4.0K Apr 12 01:10 bin
            drwxr-xr-x  4 root root  80K May  2 06:44 dev
            drwxr-xr-x 40 root root 4.0K May  2 06:44 etc
            drwxr-xr-x  2 root root 4.0K Sep 23  2011 home
            :
            ```

            * ENTRYPOINT 使用時

            ```
            # exec params command using ENTRYPOINT
            from centos
            maintainer hoge

            ENTRYPOINT ["ls", "-l"]
            ```
            ```
            # コマンド引数指定なし
            % docker run --rm IMAGE_ID
            dr-xr-xr-x  2 root root  4096 Apr 12 01:10 bin
            drwxr-xr-x  4 root root 81920 May  2 06:49 dev
            drwxr-xr-x 40 root root  4096 May  2 06:49 etc
            drwxr-xr-x  2 root root  4096 Sep 23  2011 home
            :
            ```
            ```
            # コマンド引数指定あり -> エラーになる = param の部分がコマンド指定した部分で上書きされている
            % docker run --rm IMAGE_ID ls -lh  # ENTRYPOINTのparam部分("-l")が "ls" で上書きされる
            ls: cannot access ls: No such file or directory
            :

            # 他のコマンドを指定してみる -> 同じくエラー
            % docker run --rm IMAGE_ID ps
            ls: cannot access ps: No such file or directory
            :

            # paramの部分だけ上書きされるなら、lsコマンドの引数として正しい値(オプション)を指定したらどうなるか？ -> オプションとして認識された
            % docker run --rm IMAGE_ID -lh
            dr-xr-xr-x  2 root root 4.0K Apr 12 01:10 bin
            drwxr-xr-x  4 root root  80K May  2 07:57 dev
            drwxr-xr-x 40 root root 4.0K May  2 07:57 etc
            drwxr-xr-x  2 root root 4.0K Sep 23  2011 home
            :
            ```

        * CMDの `CMD ["param1","param2"]`形式について、 __この形式の引数はENTRYPOINTで指定したコマンドに渡される__
            * この引数群は、docker run時に引数を与えた場合、その内容で上書きされる
            * つまり __この形式で与えた引数は docker run で何も引数指定しなかった場合のENTRYPOINT用デフォルト引数__ となる
            * CMDとENTRYPOINTの併用はちょくちょく見かけるテクニック

            ```
            CMD ["/usr/bin/wc","-l"] # CMD ["executable","param1","param2"] 形式
            CMD /usr/bin/wc -l  # CMD command param1 param2 形式。
            ```
            ```
            % docker run <image> -d  # <= この <image> にENTRYPOINTの指定がある場合、-dオプションはその指定コマンドに渡される

            # コマンド文字列を直接指定する場合
            ENTRYPOINT wc -l -  # ENTRYPOINT command param1 param2 形式

            # CMDとENTRYPOINTを併用する場合
            ENTRYPOINT ["/usr/bin/wc"]  # ENTRYPOINT ["executable", "param1", "param2"] 形式
            CMD ["-l", "-"]  # # CMD ["param1","param2"] 形式の記述で指定した引数は、"ENTRYPOINT command" で指定したコマンドに渡される
            ```
            ```
            # docker runで
            #   引数を指定した場合はそれをENTRYPOINTに渡して実行し（CMDで指定した引数の上書き）
            #   引数を指定しなかった場合はCMDで指定した（デフォルト）引数をENTRYPOINTに渡して実行する
            ENTRYPOINT ["/usr/bin/rethinkdb"]
            CMD ["--help"]
            ```

    * `VOLUME ["/data"]` - マウントポイント作成。ホストマシンや他コンテナのボリュームを指定
    * `USER user` - イメージ実行ユーザーを指定
    * `WORKDIR /path/to/workdir` - RUN, CMD, ENTRYPOINTのコマンド実行時の作業ディレクトリを指定
        * 1つのDockerfile内に複数指定可能
    * `ONBUILD [INSTRUCTION]` - Dockerfileビルド中に実行したいINSTRUCTIONを指定する
        * 使用例としては、コンテナのビルドと同タイミングでアプリのビルドも行いたい場合 とか

* イメージ作成、コンテナ実行の単純例

```
# Dockerfileを作成する
$ vi Dockerfile
```
```
FROM centos
MAINTAINER Niku Dorei

RUN yum -y update
RUN yum -y upgrade

CMD echo "This is a test container OK."
```
```
# ビルドする
$ docker build -t test .

Uploading context  2.56 kB
Uploading context
Step 0 : FROM centos
Pulling repository centos
539c0211cd76: Download complete
 ---> 539c0211cd76
Step 1 : MAINTAINER Niku Dorei
 ---> Running in dce9c9073ad9
 ---> c7a5250cf4b7
Step 2 : RUN yum -y update
 ---> Running in 1825a69c324d
Loaded plugins: fastestmirror
Setting up Update Process
Resolving Dependencies
--> Running transaction check
---> Package bash.x86_64 0:4.1.2-14.el6 will be updated
:
:
Complete!
 ---> a19599e4ae5b
Step 3 : RUN yum -y upgrade
 ---> Running in efcc671d114a
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.fairway.ne.jp
 * extras: mirror.fairway.ne.jp
 * updates: mirror.fairway.ne.jp
Setting up Upgrade Process
No Packages marked for Update
 ---> 293ec7165279
Step 4 : CMD echo "This is a test container OK."
 ---> Running in 0e08f17723b8
 ---> d93a7cf85eb1
Successfully built d93a7cf85eb1

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
test                latest              d93a7cf85eb1        About a minute ago   602.6 MB
```
```
# ビルドしたイメージを使ってコンテナを実行する
$ docker run d93a7cf85eb1

This is a test container OK.
```

## API

* [Remote API - Docker Documentation](http://docs.docker.io/en/latest/reference/api/docker_remote_api/)

## dockerコマンド 基本操作リファレンス
[Command Line Interface - Docker Documentation](http://docs.docker.io/en/latest/reference/commandline/cli/)

### グローバルオプション
* `-H` - デフォルト以外のIP/ポートをLISTENしてdockerデーモンを実行する ※tcp://host:port to bind/connect to or unix://path/to/socket to use
    * __docker__ groupに属するユーザーにroot同様の権限を与えることになるのでセキュリティ注意
    * `tcp://host:4243` - tcp connection on host:4243
    * `unix://path/to/socket` - unix socket located at path/to/socket

### dockerコマンド

* `docker pull NAME` - イメージのダウンロード, 完了したら`docker images`で表示される・`docker run`で使用可能になる
    * `-t` - タグを指定 
* `docker images` - インストール済のイメージ一覧
    * `-t` - ツリー形式で表示
    * `-a` - 全てのイメージを表示
    * `-q` - イメージのIDだけを表示
* `docker info` - dockerの情報表示
* `docker inspect CONTAINER|IMAGE [CONTAINER|IMAGE...]` - イメージ/コンテナの情報を __デフォルトではJSON形式で__ 表示
    * `-f` - フォーマット指定
* `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`- コンテナ作成/コマンド実行。 __コマンドの戻り値がCONTAINER ID（ハッシュ）__
    * [Docker Run Reference - Docker Documentation](http://docs.docker.io/en/latest/reference/run/)
    * `-i` - シェルにログイン
    * `-d` - デタッチモード（バックグラウンドモード）で実行。デーモンのような挙動イメージ
    * `-t` - pseudo-ttyを割り当て  [ssh経由でリモートホストで実行してるプロセスにSIGINT送りたい時 - As a Futurist...](http://blog.riywo.com/2011/04/17/005528)
    * `--name=""` - コンテナに名前を付ける
    * `-c` - CPU共有
    * `-m [<number><optional unit>]` - メモリの上限を指定。unit = b, k, m or g
    * `-h [hostname]` - ホスト名設定
    * `-p [port]` - ポートバインディング指定 (ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort)
    * `--expose [port]` - コンテナ間通信で開放するポートを指定。 __ポートバインディングと違ってホストマシンに公開はしない__
    * `-P` - 使用する全てのポートをホストマシン（のインタフェース）にバインディングする
    * `-v [host-path]:[container-path]:[rw|ro]` - ディレクトリをマウントする。ホストマシン=>コンテナ, コンテナ=>コンテナ, 共に可能 (from the host: -v /host:/container, from docker: -v /container)。最後のスイッチはrw=読み書き, ro=読み取り専用
    * `--volumes-from=[CONTAINERS|NAMES]` - コンテナを指定して、そのコンテナの共有ディレクトリをマウントする
    * `-w` - コンテナでの作業ディレクトリを指定する
    * `-e` - 環境変数を設定する
    * `--rm` - コンテナ終了時にそのコンテナを削除する
    * IMAGE はID指定でもいいし `REPOSITORY[:TAG]` 形式でも指定可能
* `docker ps [OPTIONS]` - 実行中のコンテナの表示
    * `-a` - 過去に実行したコンテナも表示
    * `-l` - 直近に実行したコンテナを表示
    * `-q` - コンテナIDだけを表示
    * `--no-trunc` - 情報を省略せずに表示
* `docker diff CONTAINER_ID` - コンテナ内のファイルシステムのdiffを表示
* `docker kill [OPTIONS] CONTAINER_ID [CONTAINER...]` - 実行中のコンテナ（のプロセス）をkillする
* `docker logs CONTAINER_ID` - コンテナで実行した操作履歴を表示する（≒historyコマンド）。
* `docker start CONTAINER [CONTAINER...]` - 停止しているコンテナを起動する
    * `-i` - シェルにログイン
* `docker stop [OPTIONS] CONTAINER [CONTAINER...]` - コンテナを停止する
    * `-t` - 停止までのwait秒数。デフォルトは10秒
* `docker attach [OPTIONS] CONTAINER` - 起動中のコンテナにアタッチ（再接続）する。 ctrl-p, ctrl-qでデタッチ可能
    * `--sig-proxy=true` - non-ttyモード。デフォルトtrueなので、ctrl-cとかで切断したいときは明示的にfalseを指定する
* `docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]` - 実行中のコンテナ（の状態）から新規イメージを作成する。後述するイメージ公開時の作法に則り `アカウント名/イメージ名` という名前（TAG）を使うとよさげ
    * `-a [author]` - author 
    * `-m [log]` - コミットログ 
    * `--run=[command json]` - このイメージを使用してコンテナを起動した際の設定（実行するコマンドとか）を指定する
        * JSONで指定、フォーマットは`docker inspect`で出力されるJSONの`Config`以降と同じ
            * 使用可能なkeyの定義は、[Command Line Interface - Docker Documentation](http://docs.docker.io/en/latest/reference/commandline/cli/#commit) とか [docker/runconfig/config.go at master · dotcloud/docker](https://github.com/dotcloud/docker/blob/master/runconfig/config.go) で確認可能
        * 「Dockerfile の設定をJSONで定義する」とほぼ同義と考えてよさげ

        ```
        $ docker commit --run='{"Cmd": ["ls","/etc"]}' test test2
        ```

* `docker search NAME` - 指定した名前（主にディストリビューション名）のイメージを検索する
* `docker build [OPTIONS] PATH | URL | -` - ソースコードやパス __=Dockerfile__ を指定して新規コンテナイメージを構築・保存する
    * `-t` - リポジトリ名（とタグ）を付加する
    * `docker build [Dockerfileのあるディレクトリ]`
    * `docker build -` - Dockerfile形式の入力を受け付ける。`<`でリダイレクトも可能
* `docker rm [OPTIONS] CONTAINER [CONTAINER...]` - コンテナを削除
    * `-l` - ベースのコンテナではなく指定されたリンクを削除
    * `-v` - コンテナに関連付けられているボリュームを削除 
* `docker rmi` - イメージを削除
* `docker export CONTAINER` - コンテナをtarにエクスポート
* `docker import URL|- [REPOSITORY[:TAG]]` - 空のファイルシステムイメージを作成し、そこにtar等から（コンテナを？）インポート
* `docker save IMAGE` - イメージをtarに保存 `docker save centos > centos.tar`
* `docker load` - tar形式のイメージを読み込み `docker load < centos.tar`
* `docker push NAME` - イメージの登録
* `docker tag [OPTIONS] IMAGE REPOSITORY[:TAG]` - イメージにタグを登録

## Docker Hub
[［速報］Docker Hub発表。ビルド、テスト、デプロイの自動化、Dockerイメージの管理など。Dockerのプラットフォーム化を推進 － Publickey](http://www.publickey1.jp/blog/14/docker_hubdockerdocker.html)

* ユーザーが作成したコンテナをアップロードして公開・共有できるサービス (旧 docker index)
* Docker Hubへのコンテナのアップロードやダウンロードなどは、すべてdockerコマンドを使って実行
    * `docker push`
    * `docker pull`
    * `docker search`
* __公開リポジトリにアップロードしたコンテナは不特定多数がダウンロード可能になるため、アップロードの際には公開したくない/公開してはいけないファイルなどが含まれていないか、注意__
* githubのリポジトリをDocker Hubへ紐付ける事が出来る
    * githubのリポジトリのルートにあるDockerfileが参照される
    * githubのリポジトリにpushがあると、上記DockerfileのビルドがDocker Hub上で走る

    ```
    Add Repository -> Automated Build
    Github をselect
    対象repositoryを選択
    Docker Hub上でのrepository nameやその他諸々を設定
    ```

* __サイトが重い__ のが難点... (docker pullとか何時間かかるんだ？ってぐらい長い時ある)


## [PENDING] dockerでsystemd
* CentOS 7 なコンテナで、systemdでサービス起動とかしたい
* 試しにapache用コンテナで`systemctl status httpd.service`してみたが、`Failed to get D-Bus connection: No connection to service manager.`というエラー発生
* ggrksしてみるとどうもDocker上ではsystemd動かんのでは？という記事がちらほら
    * [docker上のCentOS7でsystemctlからのmariadbの起動は出来ない - Qiita](http://qiita.com/tukiyo3/items/b24e4dc62de59d4a7570)
    * [Running systemctl in container fails with "Failed to get D-Bus connection" · Issue #2296 · docker/docker](https://github.com/docker/docker/issues/2296)
    * [Failed to get D-Bus connection: No connection to service manager - CentOS 7 · Issue #7459 · docker/docker](https://github.com/docker/docker/issues/7459)
    * [Bug 1033604 – Unable to start systemd service in Docker container](https://bugzilla.redhat.com/show_bug.cgi?id=1033604)
* 頑張ったぽい記事もある
    * [Running systemd within a docker container | My Blog](http://rhatdan.wordpress.com/2014/04/30/running-systemd-within-a-docker-container/)
