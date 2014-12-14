# AWS

## EC2 無料お試し
[わずか5分!? AWSのEC2でクラウドなウェブサーバーを構築してみた | 株式会社LIG](http://liginc.co.jp/web/programming/server/39969) を参考

### アカウント作成

* http://aws.amazon.com/jp/register-flow/ から


### インスタンス作成

1. https://console.aws.amazon.com/console/home?# 管理コンソール開く
2. 画面右上真ん中のプルダウンメニューからリージョンを適宜変更（デフォルトはOregon）
3. EC2 をクリック
4. Launch Instanceをクリック
5. Step 1: Choose an Amazon Machine Image (AMI)、Amazon Linux AMI 2013.09.2
6. Step 2: Choose an Instance Type、無料お試しなんでt1.microのままで
7. Step 3: Configure Instance Details、いじらない
8. Step 4: Add Storage、いじらない
9. Step 5: Tag Instance、任意のNameを入力
10. Step 6: Configure Security Group、  Create a new security group でデフォルトのSSH(22)のみ設定 を適用する。group nameは "al-secgrp-sshonly" とかで
11. reviewで確認してlaunch
12. create a new key pairする。key pair nameは ec2key とか適当にしてダウンロード。その後Launch Instances
13. view instancesで作成したインスタンスを確認

### 起動

* EC2ダッシュボードのInstancesからインスタンス一覧を表示、目的のインスタンスを選択してActionsからstartを実行

### ssh

* インスタンス作成時にダウンロードした秘密鍵を使って接続
* ホスト名は固定ではないので再起動の度に要変更

### 初期設定諸々

* とりあえず`yum -y update`
* rootのパスワード設定  ※必要なら（ノーパスsudo出来るので普通は不要のはず）

```bash
$ sudo passwd
```

* zsh, git, ctagsを入れる

```bash
$ sudo yum -y install zsh git
```

* ec2-userのシェルをzshに変える

```bash
$ sudo chsh ec2-user
ec2-user のシェルを変更します。
新しいシェル [/bin/bash]: /bin/zsh
シェルを変更しました。
```

* stowを入れる  ※dotfilesをstowで管理しているので

```bash
$ wget http://ftp.gnu.org/gnu/stow/stow-2.2.0.tar.gz
$ tar zxf stow-2.2.0.tar.gz
$ cd stow-2.2.0
$ ./configure --prefix=/usr/local
$ make
$ sudo make install
```

* 自前のdotfilesを適用する

```bash
$ git clone git@github.com:goldeneggg/dotfiles.git
$ cd dotfiles
$ ./setup.sh -L
$ git submodule update --init --recursive

$ vi vim-linux/.vimrc.common
:NeoBundleInstall
```

* ~/.gitconfigを用意する

```bash
$ vi ~/.gitconfig
[user]
    name = goldeneggg
    mail = jpshadowapps@gmail.com
[color]
    ui = auto
[core]
    excludesfile = $HOME/.gitignore
    autocrlf = input

$ cat ~/.gitconfig.aliases >> ~/.gitconfig
```

* sudo時にPATHを引き継ぐ

```bash
$ sudo visudo
```
```diff
@@ -76,6 +76,7 @@
 Defaults    env_keep += "LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
 Defaults    env_keep += "LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
 Defaults    env_keep += "LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"
+Defaults    env_keep += "PATH"

 #
 # Adding HOME to env_keep may enable a user to run unrestricted
@@ -83,7 +84,7 @@
 #
 # Defaults   env_keep += "HOME"

-Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
+#Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
```

* タイムゾーンを変える

```bash
$ sudo rm /etc/localtime
$ sudo ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

$ date
```

* localeを変える

```bash
$ sudo vi /etc/sysconfig/i18n
LANG="ja_JP.utf8"

$ locale
```

* gnu screen 4.2をインストールする

```bash
$ sudo yum -y install "gcc*" make ncurses-devel

$ mkdir src
$ cd src
$ wget https://ftp.gnu.org/gnu/screen/screen-4.2.1.tar.gz
$ tar -zxf screen-4.2.1.tar.gz
$ cd screen-4.2.1
$ ./configure --prefix=/usr/local --enable-colors256 
$ make
$ sudo make install
```


### サーバーモニタリング


### cronの設定


### Docker動く

* cgroup入ってないから無理だった。けど

```bash
$ sudo yum -y install docker
$ sudo /etc/init.d/docker start
Starting cgconfig service: Error: cannot mount hugetlb to /cgroup/hugetlb: No such file or directory
/sbin/cgconfigparser; error loading /etc/cgconfig.conf: Cgroup mounting failed
Failed to parse /etc/cgconfig.conf                         [FAILED]
Starting docker:                                           [  OK  ]
```

* [Amazon Web Services ブログ: 【AWS発表】Amazon Linux AMI 2014.03 が利用可能に](http://aws.typepad.com/aws_japan/2014/03/amazon-linux-ami-201403-is-now-available.html) このバージョンで対応された模様


### Hubot

* node.jsのインストール

```bash
$ wget http://nodejs.org/dist/v0.10.31/node-v0.10.31-linux-x64.tar.gz
$ tar zxf node-v0.10.31-linux-x64.tar.gz
$ cd tar zxf node-v0.10.31-linux-x64
$ sudo cp -r bin include lib share /usr/local/
```

* coffeescriptのインストール

```bash
$ sudo npm install -g coffee-script
```

* hubotのインストール

```bash
$ sudo npm install -g hubot
```

* bot作成

```bash
$ mkdir ~/hubot
$ cd ~/hubot
$ hubot --create ec2bot
$ cd ec2bot
$ npm install
```

* redis無効化

```
$ vi hubot-scripts.json
```
```diff
@@ -1 +1 @@
-["redis-brain.coffee", "shipit.coffee"]
+[ "shipit.coffee"]
```

* 動作確認

```bash
$ bin/hubot [--name hoge]

Hubot> hubot ping
Hubot> PONG


# HTTP Listenerのポートを明示する場合はこう
$ export PORT=9999; bin/hubot [--name hoge]
```

### コンソール上からのセキュリティ設定
* NETWORK & SECURITY => Security Groups => Create Securty Group から追加する
* Inbound - 外部からのアクセス許可設定
* Outbound - 外部へのアクセス許可設定


#### EC2上のHubotとSlackの連携

* hubot-slackのインストール

```
$ cd ~/hubot/ec2bot
$ npm install hubot-slack --save
```

* Slack側でIntegrationsを設定
    * https://<YOUR_DOMAIN>.slack.com/services/new から設定
    * Setup Instructions - 下記環境変数を確認
        * `HUBOT_SLACK_TOKEN=<YOUR_TOKEN>`
        * `HUBOT_SLACK_TEAM=<YOUR_DOMAIN>`
        * `HUBOT_SLACK_BOTNAME=slackbot`  ※ 変更可
    * Service Configuration - `http://<EC2のドメイン or IP>:<HTTP Listenerで使用するポート>`を設定する
* AWSのSecurity Group設定
    * Inbound - `<HTTP Listenerで使用するポート>`のアクセスを許可する
    * Outbound - 全開放
* hubotを環境変数`PORT=<HTTP Listenerのポート>`を設定して起動（ポートはSecurity Groupの設定で許可したポートと合わせる）

```bash
$ export HUBOT_SLACK_TOKEN=<YOUR_TOKEN>
$ export HUBOT_SLACK_TEAM=<YOUR_DOMAIN>
$ export HUBOT_SLACK_BOTNAME=<BOT_NAME>
$ export PORT=<HTTP Listenerで使用するポート>

$ export HUBOT_SLACK_CHANNELMODE=whitelist
$ export HUBOT_SLACK_CHANNELS=<JOIN_CHANNELS>  # joinするチャンネルの指定があれば指定、複数ある時はカンマ区切り

$ cd hubot/ec2bot
$ bin/hubot --adapter slack --name=<BOT_NAME> &  # バックグラウンド起動
```

* Slackで`ping`等を打って動作確認
* ブラウザから`http://<ec2 domain>:<HTTP Listenerのポート>/hubot/version`にアクセスしてhubotのバージョンが表示されるか確認



### 使用状況確認

* https://portal.aws.amazon.com/gp/aws/developer/account/index.html?action=usage-report

### Amazon Linux AMIについて


## Docker on CoreOS



## AWSの各サービス
* Elastic Beanstalk (AWS Application Container)
    * 簡単に、AWSクラウド内でアプリケーションをデプロイおよび管理することができます。
    * アプリケーションをアップロードすると、Elastic Beanstalkはキャパシティ(Amazon EC2インスタンス)をデプロイ、監視、スケールし、ロードバランサーを使って、正常に稼働しているインスタンスにリクエストを分散してくれます。
    * __要はPaas__ のようなもの
    * Docker対応済
* S3 (Scalable Storage in the Cloud)
* Route 53 (Scalable Domain Name System)
* VPS (Isolated Cloud Resources)
* CloudFront (Global Content Delivery Network)
* RDS (Managed Relational Database System)
* Redshift (Managed Petabyte-Scala Data Warehouse Service)
