# Berkshelf

「VM稼働マシンに必要なライブラリさえインストールしておけば、 __コマンド数発で欲しいVMを誰でも・何処でも作成できる__ ようにしちゃいましょう」ツール

## インストール

```
% gem install berkshelf
```

## Berksfile

* gemでいうところの Gemfile 的な存在
* `source` : 処理のコマンド 的なものの呼称。主なsourceは下記
    * `metadata` : Gemfileでいうところのgemspec的な存在。cookbookに含まれるmetadata.rb

## 主なcase study

### 既存のcookbook管理に使いたい

* 既存のcookbookをberksの管理下に置きたい

```
% berks init cookbooks
      create  cookbooks/Berksfile
      create  cookbooks/Thorfile
      create  cookbooks/.gitignore
         run  git init from "./cookbooks"
      create  cookbooks/Gemfile
      create  cookbooks/Vagrantfile
Successfully initialized
```

* berksの管理下に新規cookbookを作成したい
    * `berks cookbook`
    * この際にcookbook用のディレクトリ(recipesとかattributes)も作成されるけど、ここにレシピを定義する用途って何だろう？

```
% berks cookbook new_hoge
      create  new_hoge/files/default
      create  new_hoge/templates/default
      create  new_hoge/attributes
      create  new_hoge/definitions
      create  new_hoge/libraries
      create  new_hoge/providers
      create  new_hoge/recipes
      create  new_hoge/resources
      create  new_hoge/recipes/default.rb
      create  new_hoge/metadata.rb
      create  new_hoge/LICENSE
      create  new_hoge/README.md
      create  new_hoge/Berksfile
      create  new_hoge/Thorfile
      create  new_hoge/chefignore
      create  new_hoge/.gitignore
         run  git init from "./new_hoge"
      create  new_hoge/Gemfile
      create  new_hoge/Vagrantfile
```

### opscodeのリポジトリで公開されてるcookbookを使いたい

* berks installはcookbookを何処にインストールしてるんだろう？
    * `~/.berkshelf`
    * これ変更したければ環境変数 `BERKSHELF_PATH` をセットする

```
% vi Berksfile

site :opscode

cookbook 'mysql'
cookbook 'nginx', '~> 0.101.5'

% berks install
Installing mysql (4.0.20) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing nginx (0.101.6) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing openssl (1.1.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing build-essential (1.4.2) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing runit (1.5.8) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing yum (3.0.4) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing yum-epel (0.2.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing bluepill (2.3.1) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing rsyslog (1.10.2) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing ohai (1.0.2) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
```

```
% git init
% git add Berksfile
% git ci -m "add Berksfile to project"
```

### vagrantと連携したい

* chefと連携は、vagrantにbuilt-inされたchefを使って行われる
* まずvagrant-berkshelfプラグインをインストール

```
% vagrant plugin install vagrant-berkshelf
```

* check_executable_overwrite': "minitar" from minitar conflicts 的なエラーが出たら [Cannot install vagrant-berkshelf plugin - Stack Overflow](http://stackoverflow.com/questions/21096659/cannot-install-vagrant-berkshelf-plugin) を参考に再トライ
  * vagrantにbuilt-inされたgemコマンドでgem install、インストール先を .vatrand.d 下にする
  * ちなみにvagrant 1.5 ではプラグイン管理でBundlerを使うようになってこの事象は発生しない（らしい）

```
% /Applications/Vagrant/embedded/bin/gem install minitar --install-dir ~/.vagrant.d/gems 
minitar's executable "minitar" conflicts with archive-tar-minitar
Overwrite the executable? [yN]  y
Successfully installed minitar-0.5.4
1 gem installed
```

* Vagrantfileでberkshelfを使用することを明示する。

```
Vagrant.configure("2") do |config|
  ...
  config.berkshelf.enabled = true
  ...
end
```

* あとはVagrantfileと同じディレクトリにBerksfileを配置して必要なルールを記述すれば、 __vagrant up, vagrant provision, vagrant destroy__ を実行した際にルールが実行される
    * chefについては、chef solo と vagrant-berkshelf-pluginに同梱されているchef client の両方が使用可能 
    * "Berksfileに記述したcookbook" に依存する処理を実行した際は、そこに記述されたlocationを優先してcookbookを探す（github上だったりlocalだったり）
        * chef soloの"cookbook_path" 設定は無視される
    * BerksfileをVagrantfileとは別ディレクトリに配置したい場合は、Vagrantfileで"config.berkshelf.berksfile_path" を指定する

### Berksfileでcookbookの取得や依存関係を記述する際の例

* 基本書式は __"cookbook {name}, {version_constraint}, {options}"__

```
cookbook "nginx"

cookbook "nginx", ">= 0.101.2"
```

* cookbookのソース取得元location
    * http://cookbooks.opscode.com/api/v1/cookbooks (default)
    * コレ以外にgit, local path, 他のcommunity siteを指定可能
    * 社内githubも可能か？ → __可能__
  
  ```
  site "http://cookbooks.opscode.com/api/v1/cookbooks"
  site :opscode
  
  chef_api "https://api.opscode.com/organizations/vialstudios", node_name: "reset", client_key: "/Users/reset/.chef/reset.pem"
  chef_api :config
  
  # 複数指定で優先度定義が可能（記述順に優先される）
  chef_api :config
  site :opscode

  cookbook "artifact", "= 0.10.0"
  ```
  
  * cookbook個別でもlocation指定可能
  
  ```
  cookbook "artifact", site: "http://cookbooks.opscode.com/api/v1/cookbooks"
  
  cookbook "artifact", site: :opscode # ":opscode" : “Opscode’s newest community API” を使用
  
  cookbook "artifact", path: "/Users/reset/code/artifact-cookbook" # ローカルcookbook path。このpath配下にmetadata.rbは必須
  
  cookbook "mysql", git: "https://github.com/opscode-cookbooks/mysql.git" # git
  cookbook "mysql", git: "https://github.com/opscode-cookbooks/mysql.git", branch: "foodcritic" # branch : ブランチ指定
  cookbook “mysql”, git: “https://github.com/opscode-cookbooks/mysql.git”, ref: “eef7e65806e7ff3bdbe148e27c447ef4a8bc3881” # ref : コミットハッシュ指定
  
  cookbook "rightscale", git: "https://github.com/rightscale/rightscale_cookbooks.git", rel: "cookbooks/rightscale" # rel : リポジトリ内のディレクトリ指定
  
  cookbook "artifact", github: "RiotGames/artifact-cookbook", tag: "0.9.8" # github : githubと明示する場合
  cookbook "keeping_secrets", github: "RiotGames/keeping_secrets-cookbook", protocol: :ssh # protocol : 接続プロトコル指定
  ```
  
* chef serverを使用している環境では、[Chef Server API — Chef Docs](http://docs.opscode.com/api_chef_server.html)経由でserverからcookbook(や依存関係定義)を取得可能
  * __chef_api キーと、:config 識別子__ で指定
  
  ```
  cookbook "artifact", chef_api: :config
  
  cookbook "artifact", chef_api: "https://api.opscode.com/organizations/vialstudios", node_name: "reset", client_key: "/Users/reset/.chef/reset.pem"
  ```

### ルールをグルーピングしてグループ単位で適用を除外したりする

* __group [key] do ... end__ ブロックで記述

```
group :solo do
  cookbook 'riot_base'
end

cookbook 'riot_base', group: 'solo' # inlineでの記述も可能
```
```
% berks install --without solo # solo でグルーピングしたルールを除外
```

## vagrant, chef solo, knife soloとの組み合わせで使用するならこういうイメージかと
berks cookbook で作成される雛形を元に実行例

### berks cookbook hoge

* VagrantfileにVM, provisionerを定義する。chefとberkshelfでprovisioningすること前提の定義で
* Vagrantfileと同じディレクトリにBerksfileを作成し、 __使用するcookbookがローカルにあったり、opscode公式にあったり、サードパーティ製だったり、そのサードパーティ製がgithub上にあったり、といったcookbookのlocation管理ルール + それらの依存関係__ を記述する
  * これらcookbook達はVM実行時(berks install時)に __ローカルpathに集約__ される。knife soloでレシピ適用する際はこのローカルpathを指定するイメージ
      * この __必要なcookbookをローカルに集約する作業がBerkshelfの主な役割の1つ__
  * Berksfileはcookbook単位でも作成可能（cookbook単位で依存関係等を定義）
* 必要であればBerksfileにcookbookなりを追加

### berks install

* 必要なcookbookをローカルに集約


### vagrant up

* vagrant upするだけで、berkshelf(とchef solo)との連携により下記処理が一気に実行される
  * cookbookの集約
  * boxファイルのダウンロード → VM作成 → VM起動
  * レシピの実行
* __なんかchef soloがねぇよ的なエラーメッセージ出るんだけど、起動は成功してる__ 。guest側のchef-soloインストール状況も問題なさげに見える。何だろう？

```
% vagrant up [--provision]

	Bringing machine 'default' up with 'virtualbox' provider...
[default] Importing base box 'Berkshelf-CentOS-6.4-x86_64'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...

[Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
[Berkshelf] You should check for a newer version of vagrant-berkshelf.
[Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
[Berkshelf] You can also join the discussion in #berkshelf on Freenode.
[Berkshelf] Updating Vagrant's berkshelf: '/Users/hoge/.berkshelf/default/vagrant/berkshelf-20140127-40050-1eocii0-default'
[Berkshelf] Using berkstest (0.1.0)
[Berkshelf] Installing mysql (4.0.20) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
[Berkshelf] Installing openssl (1.1.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
[Berkshelf] Installing build-essential (1.4.2) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'

[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] Running 'pre-boot' VM customizations...
[default] Booting VM...
[default] Waiting for machine to boot. This may take a few minutes...
[default] Machine booted and ready!
[default] Setting hostname...
[default] Configuring and enabling network interfaces...
[default] Mounting shared folders...
[default] -- /vagrant
[default] -- /tmp/vagrant-chef-1/chef-solo-1/cookbooks
[default] Running provisioner: chef_solo...

The chef binary (either `chef-solo` or `chef-client`) was not found on
the VM and is required for chef provisioning. Please verify that chef
is installed and that the binary is available on the PATH.
```

```
[vagrant@berkstest-berkshelf ~]$ ls -la /tmp/vagrant-chef-1/chef-solo-1/cookbooks/
total 4
drwxr-xr-x 1 vagrant vagrant  204 Jan 27 06:00 .
drwxr-xr-x 3 root    root    4096 Jan 27 06:01 ..
drwxr-xr-x 1 vagrant vagrant  646 Jan 27 06:00 berkstest
drwxr-xr-x 1 vagrant vagrant  272 Jan 27 06:00 build-essential
drwxr-xr-x 1 vagrant vagrant  340 Jan 27 06:00 mysql
drwxr-xr-x 1 vagrant vagrant  272 Jan 27 06:00 openssl
```

* [vagrant-omnibusで簡単Chef Client/Chef Soloインストール | Ryuzee.com](http://www.ryuzee.com/contents/blog/6651) を参考に vagrant-omnibus を導入してみる → __エラー解消__

```
% vagrant plugin install vagrant-omnibus

% vagrant up
:
(略)
:
[default] -- /tmp/vagrant-chef-1/chef-solo-1/cookbooks
[default] Installing Chef 11.8.2 Omnibus package...
[default] Downloading Chef 11.8.2 for el...
[default] downloading https://www.opscode.com/chef/metadata?v=11.8.2&prerelease=false&p=el&pv=6&m=x86_64
  to file /tmp/install.sh.2544/metadata.txt
trying curl...
[default] url   https://opscode-omnibus-packages.s3.amazonaws.com/el/6/x86_64/chef-11.8.2-1.el6.x86_64.rpm
md5     10f3d0da82efa973fe91cc24a6a74549
sha256  044558f38d25bbf75dbd5790ccce892a38e5e9f2a091ed55367ab914fbd1cfed
[default] downloaded metadata file looks valid...
[default] downloading https://opscode-omnibus-packages.s3.amazonaws.com/el/6/x86_64/chef-11.8.2-1.el6.x86_64.rpm
  to file /tmp/install.sh.2544/chef-11.8.2.x86_64.rpm
trying curl...
[default] Checksum compare with sha256sum succeeded.
[default] Installing Chef 11.8.2
installing with rpm...
[default] warning:
[default] /tmp/install.sh.2544/chef-11.8.2.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
[default] Preparing...
[default] ##################################################s
[default]
[default] chef
[default] #
[default] #
:
(略)
:
[default] Thank you for installing Chef!
[default] Running provisioner: chef_solo...
Generating chef JSON and uploading...
Running chef-solo...
[2014-01-27T06:47:14+00:00] INFO: Forking chef instance to converge...
[2014-01-27T06:47:14+00:00] INFO: *** Chef 11.8.2 ***
[2014-01-27T06:47:14+00:00] INFO: Chef-client pid: 2844
[2014-01-27T06:47:14+00:00] INFO: Setting the run_list to ["recipe[berkstest::default]"] from JSON
[2014-01-27T06:47:14+00:00] INFO: Run List is [recipe[berkstest::default]]
[2014-01-27T06:47:14+00:00] INFO: Run List expands to [berkstest::default]
[2014-01-27T06:47:14+00:00] INFO: Starting Chef Run for berkstest-berkshelf
[2014-01-27T06:47:14+00:00] INFO: Running start handlers
[2014-01-27T06:47:14+00:00] INFO: Start handlers complete.
[2014-01-27T06:47:14+00:00] INFO: Chef Run complete in 0.071216636 seconds
[2014-01-27T06:47:14+00:00] INFO: Running report handlers
[2014-01-27T06:47:14+00:00] INFO: Report handlers complete
[2014-01-27T06:47:14+00:00] INFO: Forking chef instance to converge...
```

### vagrant provision

* 適用するレシピを追加・修正した際は、vagrant provision で適用する
  * mysql::server を適用してみる
  * 該当のレシピが未取得であればまず berks install する

```
% vi Vagrantfile
    chef.run_list = [
        "recipe[mysql::server]"
    ]


% vagrant provision
[Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
[Berkshelf] You should check for a newer version of vagrant-berkshelf.
[Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
[Berkshelf] You can also join the discussion in #berkshelf on Freenode.
[Berkshelf] Updating Vagrant's berkshelf: '/Users/hoge/.berkshelf/default/vagrant/berkshelf-20140127-66259-11mmpb-default'
[Berkshelf] Using berkstest (0.1.0)
[Berkshelf] Using mysql (4.0.20)
[Berkshelf] Using openssl (1.1.0)
[Berkshelf] Using build-essential (1.4.2)
[default] Chef 11.8.2 Omnibus package is already installed.
[default] Running provisioner: chef_solo...
Generating chef JSON and uploading...
Running chef-solo...
[2014-01-27T07:01:15+00:00] INFO: Forking chef instance to converge...
[2014-01-27T07:01:15+00:00] INFO: *** Chef 11.8.2 ***
[2014-01-27T07:01:15+00:00] INFO: Chef-client pid: 2503
[2014-01-27T07:01:15+00:00] INFO: Setting the run_list to ["recipe[mysql::server]"] from JSON
[2014-01-27T07:01:15+00:00] INFO: Run List is [recipe[mysql::server]]
[2014-01-27T07:01:15+00:00] INFO: Run List expands to [mysql::server]
[2014-01-27T07:01:15+00:00] INFO: Starting Chef Run for berkstest-berkshelf
[2014-01-27T07:01:15+00:00] INFO: Running start handlers
[2014-01-27T07:01:15+00:00] INFO: Start handlers complete.
[2014-01-27T07:01:15+00:00] WARN: Cloning resource attributes for directory[/var/lib/mysql] from prior resource (CHEF-3694)
[2014-01-27T07:01:15+00:00] WARN: Previous directory[/var/lib/mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/_server_rhel.rb:11:in `block in from_file'
[2014-01-27T07:01:15+00:00] WARN: Current  directory[/var/lib/mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/_server_rhel.rb:20:in `from_file'
[2014-01-27T07:01:55+00:00] INFO: package[mysql-server] installing mysql-server-5.1.71-1.el6 from base repository
[2014-01-27T07:02:11+00:00] INFO: directory[/var/log/mysql] created directory /var/log/mysql
[2014-01-27T07:02:11+00:00] INFO: directory[/var/log/mysql] owner changed to 27
[2014-01-27T07:02:11+00:00] INFO: directory[/var/log/mysql] group changed to 27
[2014-01-27T07:02:11+00:00] INFO: directory[/var/log/mysql] mode changed to 755
[2014-01-27T07:02:11+00:00] INFO: directory[/etc/mysql/conf.d] created directory /etc/mysql/conf.d
[2014-01-27T07:02:11+00:00] INFO: directory[/etc/mysql/conf.d] owner changed to 27
[2014-01-27T07:02:11+00:00] INFO: directory[/etc/mysql/conf.d] group changed to 27
[2014-01-27T07:02:11+00:00] INFO: directory[/etc/mysql/conf.d] mode changed to 755
[2014-01-27T07:02:11+00:00] INFO: template[initial-my.cnf] backed up to /var/chef/backup/etc/my.cnf.chef-20140127070211.686264
[2014-01-27T07:02:11+00:00] INFO: template[initial-my.cnf] updated file contents /etc/my.cnf
[2014-01-27T07:02:11+00:00] INFO: template[initial-my.cnf] sending start action to service[mysql-start] (immediate)
[2014-01-27T07:02:13+00:00] INFO: service[mysql-start] started
[2014-01-27T07:02:13+00:00] INFO: execute[assign-root-password] ran successfully
[2014-01-27T07:02:13+00:00] INFO: template[/etc/mysql_grants.sql] created file /etc/mysql_grants.sql
[2014-01-27T07:02:13+00:00] INFO: template[/etc/mysql_grants.sql] updated file contents /etc/mysql_grants.sql
[2014-01-27T07:02:13+00:00] INFO: template[/etc/mysql_grants.sql] owner changed to 0
[2014-01-27T07:02:13+00:00] INFO: template[/etc/mysql_grants.sql] group changed to 0
[2014-01-27T07:02:13+00:00] INFO: template[/etc/mysql_grants.sql] mode changed to 600
[2014-01-27T07:02:13+00:00] INFO: template[/etc/mysql_grants.sql] sending run action to execute[install-grants] (immediate)
[2014-01-27T07:02:13+00:00] INFO: execute[install-grants] ran successfully
[2014-01-27T07:02:13+00:00] INFO: execute[install-grants] sending restart action to service[mysql] (immediate)
[2014-01-27T07:02:19+00:00] INFO: service[mysql] restarted
[2014-01-27T07:02:19+00:00] INFO: service[mysql] enabled
[2014-01-27T07:02:19+00:00] INFO: Chef Run complete in 63.882187996 seconds
[2014-01-27T07:02:19+00:00] INFO: Running report handlers
[2014-01-27T07:02:19+00:00] INFO: Report handlers complete
[2014-01-27T07:01:15+00:00] INFO: Forking chef instance to converge...
```
```
# ゲスト側

[vagrant@berkstest-berkshelf ~]$ which mysql
/usr/bin/mysql
[vagrant@berkstest-berkshelf ~]$ mysql --version
mysql  Ver 14.14 Distrib 5.1.71, for redhat-linux-gnu (x86_64) using readline 5.1
```
