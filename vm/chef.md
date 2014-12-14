# Chef

## Chefの情報はとりあえずココみとけ
* chefはプロビジョニングツール
* http://docs.opscode.com/#id3


## Chef Soloとは？
* Chef のスタンドアロン版、Chef サーバがなくてもノードのローカルで動作する
* http://wiki.opscode.com/display/chef/Chef+Solo
* http://wiki.opscode.com/display/~tily/Chef+Solo (日本語)

### Knife Soloとは？

> chef-soloはChef Serverが不要であるため気軽に利用できるが、設定対象のそれぞれのサーバーにChef ClientやCookbookを配置して管理しなければならない。

> この作業を自動化するツールが、今回紹介するknife-soloだ。

> knife-soloは、作業用マシン上に用意したCookbookなどの設定ファイルを設定対象のマシンにコピーし、

> 設定対象のマシン上でchef-soloコマンドを実行する、という一連の作業を自動実行するツールだ

## インストール
* Ruby 2.0.0 を導入済みであることが前提
  * オイラはrbenv on Mac な環境で実施
* **対象バージョンはこちら**。これを gem install hoge する

```
% gem list -l | grep -e chef -e knife
chef (11.8.0)
chef-zero (1.7.2)
knife-solo (0.4.0)
knife-solo_data_bag (0.4.0)
```

### chef

```
$ gem install chef
$ knife configure  # とりま全部デフォルトで
WARNING: No knife configuration file found
Where should I put the config file? [/Users/golden_eggg/.chef/knife.rb]
Please enter the chef server URL: [http://mypc.com]
Please enter an existing username or clientname for the API: [hoge]
Please enter the validation clientname: [chef-validator]
Please enter the location of the validation key: [/etc/chef/validation.pem]
Please enter the path to a chef repository (or leave blank):
*****

You must place your client key in:
  /Users/hoge/.chef/hoge.pem
Before running commands with Knife!

*****

You must place your validation key in:
  /etc/chef/validation.pem
Before generating instance data with Knife!

*****
Configuration file written to /Users/hoge/.chef/knife.rb
```

* **knifeコマンドには鍵(pem)が必要なものがある。これを動かしたければopscodeのcommunityに登録して鍵をダウンロードしておく必要あり**
  * http://docs.opscode.com/config_rb_knife.html 
  * .chef/knife.rbの "client_key"
* **".chef"ディレクトリ（とknife.rb）は、下記優先順位で読み込まれる模様。雛形ディレクトリの下に作っておくほうが無難かも（後で色々ハマるから）**
  * [雛形ディレクトリ]/.chef
  * $HOME/.chef
  
  ```
  The configuration file is located at: ~/.chef/knife.rb. If a knife.rb file is present in the .chef/knife.rb directory in the chef-repo, the settings contained within that file will override the default configuration settings.
  ```

### Knife Solo（とChef Soloの連携）

```
$ gem install knife-solo
$ (rbenv rehash)

$ knife solo init provisioning

% ls -la
total 8
drwxr-xr-x  10 hoge  INTRA\Domain Users  340 11 27 17:20 ./
drwxr-xr-x   5 hoge  INTRA\Domain Users  170 11 27 17:20 ../
drwxr-xr-x   3 hoge  INTRA\Domain Users  102 11 27 17:20 .chef/
-rw-r--r--   1 hoge  INTRA\Domain Users   12 11 27 17:20 .gitignore
drwxr-xr-x   3 hoge  INTRA\Domain Users  102 11 27 17:20 cookbooks/
drwxr-xr-x   3 hoge  INTRA\Domain Users  102 11 27 17:20 data_bags/
drwxr-xr-x   3 hoge  INTRA\Domain Users  102 11 27 17:20 environments/
drwxr-xr-x   3 hoge  INTRA\Domain Users  102 11 27 17:20 nodes/
drwxr-xr-x   3 hoge  INTRA\Domain Users  102 11 27 17:20 roles/
drwxr-xr-x   3 hoge  INTRA\Domain Users  102 11 27 17:20 site-cookbooks/


% cat .chef/knife.rb
cookbook_path    ["cookbooks", "site-cookbooks"]
node_path        "nodes"
role_path        "roles"
environment_path "environments"
data_bag_path    "data_bags"
#encrypted_data_bag_secret "data_bag_key"

knife[:berkshelf_path] = "cookbooks"
```

#### guest OSにchef-solo実行環境構築

```
$ vi ~/.ssh/config
Host 192.168.56.*
 User vagrant
 IdentityFile ~/.vagrant.d/insecure_private_key
 
$ vagrant up
$ cd chef-repo
$ knife solo prepare vagrant@192.168.56.21
WARNING: No knife configuration file found
Bootstrapping Chef...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
101  6471  101  6471    0     0   1082      0  0:00:05  0:00:05 --:--:-- 38064
Downloading Chef  for el...
Installing Chef
warning: /tmp/tmp.dMHpbX6N/chef-.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
Preparing...                ########################################### [100%]
   1:chef                   ########################################### [100%]
Thank you for installing Chef!
Generating node config 'nodes/192.168.56.21.json'...
```

* knife solo prepare でエラーが発生した場合の対処
  * "ERROR: ArgumentError: Could not parse PKey: no start line"
    * vmにsshするときに使用する秘密鍵とペアになる公開鍵を作成すると解決するらしい（何故？）
  * "HostKeyMismatch"
    * 一旦ホストマシンのknown_hostsからvagrantサーバのエントリを削除してみる

### vagrantとの連携
* **よくある事例は、knife solo initした雛形ディレクトリにVagrantfileも置くって例**
  * 役割分担が若干被ってたりしてよく分からんところはあるが。。。

## Chefの構成要素
* http://wiki.opscode.com/pages/viewpage.action?pageId=24019634 (日本語)
* http://wiki.opscode.com/pages/viewpage.action?pageId=24019711 (日本語 補足的なページ)

### Cookbooks
* http://docs.opscode.com/essentials_cookbooks.html
* Chefでの**配布の基本となる単位**
  * githubで公開されているレシピもこの単位が基本
* システムを設定するために利用するレシピ、リソース定義、属性、ライブラリ、クックブック・ファイル、テンプレートと、メタデータを集めたもの
* 1つのパッケージまたはサービスを設定するもの　という単位でまとめられる
  * たとえば MySQL のクックブックにはクライアント・サーバ両方のレシピが含まれ、さらには属性ファイルの中には、設定可能な値にまともなデフォルト値が割り当てられる


#### attributes
* http://docs.opscode.com/essentials_cookbook_attribute_files.html
* 属性値・設定値を定義する
* attributeの主なタイプ↓ : こちらも参考に http://docs.opscode.com/chef_overview_attributes.html
  * default - 
  * force_default - 
  * normal - 
  * override - 
  * force_override - 
  * automatic - 
 
```ruby
# attributes/<filename>.rb
default['hoge'] = 'huga'  # "default[<name>] = <value>" で設定
 
# recipes/default.rb
node.default['hoge'] # nodeオブジェクト "default" で設定値を参照
node['hoge'] # ".default" は省略してもよさげ。　attribute/<sub_dir>/<filename>.rb という風にsub_dirを挟んで配置すると参照エラーになる
```

* (Ohaiによって)自動設定されるattribute達もあって、recipeとかの中で使用可能
  * "node['platform']"
  * "node['platform_version']"
  * "node['ipaddress']"
  * "node['macaddress']"
  * "node['fqdn']"
  * "node['domain']"
  * "node['recipes']"
  * "node['roles']"
  * "node['ohai_time']"
  * **これもvagrant使ってたらvagrantと担当範囲被ってる感があるが...**

#### definitions
* http://docs.opscode.com/essentials_cookbook_definitions.html
* 新しいResourceを自分で作れる仕組み
* definitionsの例をWebから拝借

```ruby
# 名前がpostgresql_func
# その後ろがデフォルト引数（設定値）
define :postgresql_func, :action => :create, :owner => "postgres" do
  contrib = node.postgresql.contrib_dir  # node.postgresql.contrib_dirはattributesで設定
 
  case params[:action]
  when :create
      execute "psql -1 -f #{contrib}/#{params[:mod]}.sql  #{params[:name]}" do
        # params[:name] はpostgres_funcの引数
        user "postgres"
        not_if "psql -At1 -c \"SELECT prosrc FROM pg_proc \" #{params[:name]} | grep '#{params[:func]}'", :user => "postgres"
    end
 
  when :drop
    execute "psql -1 -f #{contrib}/uninstall_#{params[:mod]}.sql
    #{params[:name]}" do
      user "postgres"
      not_if "psql -At1 -c \"SELECT prosrc FROM pg_proc \" #{params[:name]} | grep '#{params[:func]}'", :user => "postgres"
    end
  end
end
 
 
# ↑こうしておけば、あとはrecipeの中でこう使えます。
postgresql_func "template1" do
  mod "dblink"
  func "dblink_connect"
end
```

#### files
* http://docs.opscode.com/essentials_cookbook_files.html
* 固定ファイル/ディレクトリを設置
* "cookbook_file" resourceでファイルを作成する際に、sourceとして使用する　とかが主な事例
  * cookbookに同梱したrpmをインストールしたい時　とか
* 公式のmuninのcookbookはcrontab配置で使用している

```ruby
# files/default/munin-cron
MAILTO=root
@reboot         root  if [ ! -d /var/run/munin ]; then /bin/bash -c 'perms=(`/usr/sbin/dpkg-statoverride --list /var/run/munin`); mkdir /var/run/munin; chown ${perms[0]:-munin}:${perms[1]:-root} /var/run/munin; chmod ${perms[2]:-0755} /var/run/munin'; fi
*/5 * * * *     munin if [ -x /usr/bin/munin-cron ]; then /usr/bin/munin-cron; fi
14 10 * * *     munin if [ -x /usr/share/munin/munin-limits ]; then /usr/share/munin/munin-limits; fi
 
 
 
# recipes/server.rb
 
  cookbook_file "/etc/cron.d/munin" do
    source "munin-cron"
    mode "0644"
    owner "root"
    group node['munin']['root']['group']
    backup 0
  end
```

#### libraries
* http://docs.opscode.com/essentials_cookbook_libraries.html
* Chefに新しいクラスやメソッドを追加することができる仕組み
  * Chef::Recipe とかの Chef で使用する Ruby Classの拡張をする とか  
* デカくなってきたときの名前空間の整理が大事
  * opscodeの例ではmoduleを使った名前空間管理が多い
  * ベースとなる名前空間は "Chef::Recipe"
      * "class Chef::Recipe::ISP" を自作した場合、Recipeからは "ISP.[class/method name]" でアクセス
* **libraries/[library_name].rb** の形式で配置する

 
#### metadata.rb
* http://docs.opscode.com/essentials_cookbook_metadata.html
* 他に依存しているクックブックがその依存関係を表現している箇所がメタデータ
  * 要はcookbookの説明etcのメタ情報
* メタデータなので記述必須というわけではない

#### recipes
* http://docs.opscode.com/essentials_cookbook_recipes.html
* **Chefでの設定の基本となる単位**
* リソースを書くファイル

##### DSL of recipes
* http://docs.opscode.com/dsl_recipe.html
 
##### resources
* **レシピで使用可能な処理(アクション？)の定義を "リソース"と呼ぶ。**パッケージのインストールだったりサービスの再起動だったりユーザーの追加だったり。。。
* **「あるマシンに何度リソースを適用したとしても結果は変わらず、常に適切に設定されたマシンを得ることができる」**というポリシー
* レシピ書くとき索引的に読むならココ : http://docs.opscode.com/resource.html
* リソースまとめページ - http://docs.opscode.com/chef/resources.html
* Lightweight Resources and Providers（カスタムresource集） - http://docs.opscode.com/lwrp.html
  * 例 : mysql - http://docs.opscode.com/lwrp_mysql.html
* **resourceの4大構成要素 - type, name, attributes, actions**

```ruby
type "name" do
   attribute "value"
   action :type_of_action
end
```

* 主なresource
  * "user" - ユーザーの管理（追加、変更、削除）
  * "group" - ユーザーの管理（追加、変更、削除）
  * "package" - パッケージの管理（インストール）
  * "remote_file" - リモートファイルの操作（ダウンロード etc）
  * "execute" - コマンドの実行
  * "script" - シェルスクリプトの実行
  * "template" -  ファイルの管理（/etc/xxx.conf 的な設定ファイルの作成、変更検知 etc）
  * "service" - サービスの管理（追加、起動、停止 etc）
  
###### Common Functionality
* http://docs.opscode.com/resource_common.html
* 全resourceは共通の action, attribute, guard/notification etc を共有している
  * actions : アクション
      * ":nothing" アクション - 何もしない。例えばある条件を満たした場合だけ実行したいリソースに指定する　とか（ファイルの変更を検知時のみ実行　とか ）
      * ":create" アクション（commonでは無いがよく見るやつ） - 作成する
      * ":install" アクション（commonでは無いがよく見るやつ） - インストールする
      * ":run" アクション（commonでは無いがよく見るやつ） - 実行する
  * attributes : 属性
      * ignore_failure - 失敗しても後続処理を継続するか（default: false）
      * provider
      * retries - 失敗時のリトライ回数（default: 0）
      * retry_delay - リトライ時のdelay（default: 2）
      * supports - どんな操作をサポートするか。"service" resourceで使用されるケースがほとんど
  * guards : 条件（not_if / only_if）
      * not_if - 指定条件の評価結果が偽(1)なら実行「not_if "シェルコマンド"」 的な書き方が一般的
      * only_if - 指定条件の評価結果が真(0)なら実行
  * notifications : 通知
      * 該当resourceに変更があった場合に、他のresourceを実行する
      * notifies <action>, "<実行対象resouce名>", <timer> という形式が一般的？
      * timer
          * :delayed - 遅延実行、**全リソースの最後**に実行される
          * :immediately - 直ちに実行

##### Providers
* resourceを実行する際にその環境に合ったものを適切に割り当てる、その割り当て先を "provider" と呼ぶ
  * パッケージのインストールならyumなのかaptなのかとか
  * 「リソースが抽象化しているものを特定のプラットフォーム向けに特化して実装したもの」

#### templates
* http://docs.opscode.com/essentials_cookbook_templates.html
* 設定ファイル(とか）のひな形
* erbで記述
* templateリソース
  * source 属性でファイル名を指定
  * varibles 属性で **変数名=>値 なハッシュ** を記述してtemplate用変数の定義が可能
      * erb内では **@変数名** で参照
      * まあtemplate内でも <%= node['hoge]['huga'] %> と書けばattributeの内容を直接参照可能なのだが
      * chefのnodesとかrolesでattributesを上書きしたい時に、いわゆる「ロード順の罠」で上書きしたい内容が反映されない、ってケースを回避するのには有効かも
* ver.11から、**renderメソッドを使ってtemplate内から別templateのinclude的な事が出来る**

```ruby
<%= render "simple.txt.erb", :variables => {:user => Etc.getlogin }, :local => true %>
```

### Data Bags
* http://docs.opscode.com/essentials_data_bags.html
* attributesっぽい。けど微妙に違う。recipeやcookbookと直接紐付いていない属性情報
* 種別を識別するDIRを掘って、その下にJSON形式で作成　※このオペレーションは "knife data bag create ..." コマンドで代替可能（詳細は公式マニュアル参照）
  * 例 : <Chef dir>/data_bags/sushi/neta.json
  * "id" は必須要素

```
$ mkdir data_bags/sushi
$ vi data_bags/sushi/neta.json
```

```
{
    "id" : "sushi_neta",
    "netas" : [
        "maguro", 
        "hamachi",
        "aji"
    ]
}
```

```
netas = data_bag_item("sushi", "sushi_neta")["netas"]
template "/tmp/sushineta" do
    owner "root"
    group "root"
    source "sushirou.erb"
    variables ({
        :neta_tachi => netas
    })
end
```

```
<% for neta in @neta_tachi %>
sushi_neta is <%= neta %>
<% end %>
```

#### 暗号化について
* **暗号化が使える**。ネットの記事でよく見るのはパスワード管理とか、user作成レシピで使うユーザー情報の定義とか
* http://docs.opscode.com/chef/essentials_data_bags.html#encrypt-a-data-bag
  * Chef::EncryptedDataBagItem.load
  * Chef::EncryptedDataBagItem.load_secret
* **chef-soloでdata_bagの暗号化を使用したい場合は、"gem install knife-solo_data_bag" して "knif solo data bag ..." コマンドを使う**

```
% gem install knife-solo_data_bag
% cd [repo dir]
% cat solo.rb
encrypted_data_bag_secret "/tmp/chef-solo/data_bag_key"  # デフォルト内容確認

% openssl rand -base64 512 > data_bag_key  # 漏洩せぬよう。。
% knife solo data bag create passwords mysql --secret-file data_bag_key
（もしココでコケたら、data_bag_pathがデフォルトの /var/chef/data_bags になってる可能性あり。knife.rbを修正するなり雛形ディレクトリに独自の.chef/knife.rb作るなりして回避する）

{
  "id": "mysql",
  "pass": "XXXXX"  # これ追記
}

% tree
.
└── passwords
    └── mysql.json
```

* 作成したdata_bagの確認

```
% cat passwords/mysql.json
{"name":"data_bag_item_passwords_mysql","json_class":"Chef::DataBagItem","chef_type":"data_bag_item","data_bag":"passwords","raw_data":{"id":"mysql","pass":{"encrypted_data":"b5PApGGm7LvtSLNSJg/01DNYa6fczUVim4Wj0Lf8IcI=\n","iv":"KwX5NyIiBU6+TV61Eudb5A==\n","version":1,"cipher":"aes-256-cbc"}}}


% knife solo data bag show passwords mysql
id:   mysql
pass:
  cipher:         aes-256-cbc
  encrypted_data: b5PApGGm7LvtSLNSJg/01DNYa6fczUVim4Wj0Lf8IcI=

  iv:             KwX5NyIiBU6+TV61Eudb5A==

  version:        1


% knife solo data bag show passwords mysql --secret-file data_bag_key
id:   mysql
pass: xxxxx  # secret-fileを指定してshowしたら、暗号化されてないパスワードが表示される
```

* 暗号化したdata_bagの内容をrecipeで使う

```
% vi nodes/xxx.xx.xx.xx.json
{
    "default_attributes" : {
        "test" : {
            "secret_key" : "data_bag_key"
        }
    },
    "run_list":[
        "recipe[test]"
    ]
}
```
```
% vi cookbooks/test/recipes/default.rb

# load_secret でsecretを取得
secret_key = Chef::EncryptedDataBagItem.load_secret(node['test']['secret_key'])

# secretを使用してdata_bagをload
passwords = Chef::EncryptedDataBagItem.load('passwords','mysql', secret_key)

Chef::Log.logger.info("@@@@@ INFO passwords=[#{passwords}]")
Chef::Log.logger.info("@@@@@ INFO passwords[pass]=[#{passwords['pass']}]") # 復号化されたpassが表示される
```

### chef-solo（の要素群）
* http://docs.opscode.com/chef_solo.html

#### Environments
* http://docs.opscode.com/essentials_environments.html
* **chef-soloでは使えない**

#### Nodes
* http://docs.opscode.com/essentials_nodes.html
* Chefクライアントを起動させるホスト
* (JSONを使うなら) **"[ホストIP].json" ってファイルに固有な/上書きする属性や、実行するレシピのリストを定義**する
* **vagrantを使う場合、Vagrantfileで定義する/出来る内容と結構重複してたりするかも。。。**

##### Run-lists
* ノードが実行するレシピの一覧
* **nodes/[host].json にjsonで記述**する（のがよくあるケース）
* "run_list" がkey、recipe[recipe名]がvalue　の形で定義
  * valueを配列にして実行したいrecipeを複数記述
  * "recipe[cookbook名]"
  * "recipe[cookbook名::recipe名]"
  
```
{
  "run_list": [
    "recipe[test]",  # cookbooks/test/recipes/default.rb を実行
    "recipe[test::test2]" # cookbooks/test/recipes/test2.rb を実行 
 ]	
}
```

#### Roles
* http://docs.opscode.com/essentials_roles.html
* ノードの中から機能の似ているものをグループとしてまとめることができる
* ノードと同じ要素から成り立っている（属性と実行リスト）
* **roles/[role名].json にjsonで記述**する（のがよくあるケース）
  * name - roleの名前、ファイル名と合わせる（合わせないと駄目？）
  * default_attributes - このroleを実行する全てのnodeに適用されるべき属性値を定義（すると良いらしい）
      * recipe個別ではなくもっとグローバルな設定とか
  * override_attributes - 既存の属性値(recipeの中の？)を上書きしたい場合に定義
      * **default_attributesによる上書きと何が違うんだっけ？**（挙動は同じに見える）
      * cookbookのattribute設定を上書き、なら同じ挙動になる。
      * **role <-> role（や、role <-> run_list）間のdefault_attributesをoverride_attributesで上書き　というケース（かな？）**
      * ↓って書いてるけど、chef-soloでもこういう挙動には見えなかったが。。。
      
      ```
      chef-soloにoverride_attributesの概念はない roleに対してはoverrideできるが、レシピに記載されているdefaultのattributeに対してchef-soloの場合効果がない override_attributesを指定しないでjsonに直接実行すれば、default_attributeの値を上書きして実行してくれる
      ```
      
  * json_class - とりあえず "Chef::Role" ベタ書きでヨサゲ
  * description - 説明。あれば書く
  * chef_type - とりあえず "role" ベタ書きでヨサゲ
  * run_list - nodeと同じく実行するレシピを一覧で記述
  
```
    {
        "name" : "webserver",
        "default_attributes" : {
          "install_java" : {  # install_java レシピの
            "jdk_version" : "7u45" # jdk_version というattributeをdefault値で上書き(?) → されたっぽい
          }
        },
        "override_attributes" : {
          "install_java" : {  # install_java レシピの
            "jdk_version" : "7u45" # jdk_version というattributeを上書き
          }
        },
        "json_class": "Chef::Role",
        "description": "",
        "chef_type": "role",
        "run_list": [
            "recipe[yum::epel]",
            "recipe[nginx]",
            "recipe[ruby]"
        ]
    }
```

* node定義でroleを使用する際は**run_listに"role[role名]" を指定する**。↓こんな感じ
  * **roleでroleを使用**も出来るんだっけ？ → 出来そう

```
{
  "run_list": [
    "role[webserver]"  # roles/webserver.json のrun_listを実行
 ]
}
```

#### Configuration Files
* http://docs.opscode.com/config.html

##### solo.rb
* recipeやroleのパスを設定
* http://docs.opscode.com/config_rb_solo.html
  * cookbook_path
  * data_bag_path - (Default = /var/chef/databags)
  * environment - (Default = /var/chef/environments)
  * environment_path
  * log_level - :debug, :info, :warn, :error
  * log_location - (Default = STDOUT)
  * solo - (Default = false)


## ツール

### Knife
* http://docs.opscode.com/knife.html

## Tips
* 「処理Aが正常にdoneしたら処理Bを実行」的なケース -> notifies をうまいこと使う
  * 特定の条件下だけ実行	したい処理(処理B)側に "action :nothing"。これ大事
* コマンドで処理したければ下記resourceを使い分ければよさげ
  * executeリソース (oneliner書くイメージ。単にコマンド一発とか)
    * command属性
  * script (スクリプトファイル書くイメージ)
    * code属性
* レシピの構文チェックは "knife cookbook test <recipe name>"
* **cookbookとroleは違う**。混同しやすいかも

```
Cookbook["apache"]のRecipeのdefaults.rbはapache本体のインストールでした。
そして、defaultではないその他のRecipeはapache modulesのインストールでした。

これで何となくCookbookが分かって来た気がしますが、さらに理解を深めるためcookbook[MySQL]を見てみました。

cookbook["MySQL"]のRecipesの中にはdefault.rb,client.rb,server.rbがありました。
Recipeのdefault.rbには一行。

include_recipe "mysql::client"
```

### 調べるTODO

#### data_bag と attribute（の上書き） の使い分け

* **data_bagは内容の暗号化が行える**
  * 管理者のみが知るべき情報の定義に向いている。**アカウント情報とかパスワードとか**
  * ＝リポジトリ管理すべきではない情報 と言える

#### 特定の条件を満たしたらフラグを立てる→フラグが立ってたら実行する処理を設ける　的なことどうやればよいか？

#### 処理の汎用化（Template Method的な）事して複数のcookbookで流用するにはどうすればよいか？

* 汎用モジュール（≒クラス群, メソッド群）は別リポジトリで管理する
* git submodule で各cookbookに取り込む
* **取り込む際は libraries ディレクトリへ** 取り込んで、自作モジュールとして使用するイメージ
  * ということは、librariesディレクトリの流儀に従うの大事

#### nodes/<ip>.json で実行するレシピを定義する際に「どのサーバでも必ず実行するrecipe」を共通化出来るような記述方法は？

#### 実行時引数　的なものを渡して処理制御したければどうすればよいか？

#### [解決]なんか上から下に順番に実行されてないように見えるんだけど気のせいか...特に変数使用時

* **http://acro-engineer.hatenablog.com/entry/2013/09/20/124837**  
  
  ```
  実を言うとChefは、Recipeを実行する前に、Recipeに記述されているリソースBlockコードを、リソースオブジェクトに変換する処理（以降、これをコンパイルということにします）を行った後に、コンパイルによって変換されたリソースオブジェクトを順番に実行しています。
  リソースBlockの外のコードは、リソースBlockのコードをリソースオブジェクトに変換するコンパイル処理時にすでに実行されてしまっているのです。
  ```
  
#### [解決] roleのjsonに設定したoverride_attributes（をattributeファイルで動的に参照するという仕組み）が効かない

* 「attributeの評価タイミングでは上書きattributeが反映されない」の件はちょっと辛い。可変設定はrecipesでやるべき？

```
chef-soloにoverride_attributesの概念はない roleに対してはoverrideできるが、レシピに記載されているdefaultのattributeに対してchef-soloの場合効果がない override_attributesを指定しないでjsonに直接実行すれば、default_attributeの値を上書きして実行してくれる
```

```
どうも、Attributesが評価される段階ではまだRoleやEnvironmentで設定した値は反映されないらしい
```

#### centos6.5にしてから、いくつかのレシピがCannot allocate memoryでコケ始めた(例:javaのインストールとか)

* 1GB → 1.5GBに拡張しても駄目
* 2GBに拡張 → **OK**
* **なんか単純に母艦のmacのメモリに余裕がなかっただけじゃないか疑惑**

#### overcommit_memoryを無効化したらどうか？ → 解決せず...

```
[root@vmes1 ~]# vi /etc/sysctl.conf
vm.overcommit_memory = 2
vm.overcommit_ratio = 99
```

## githubに上がってるレシピ
* https://github.com/treasure-data/chef-td-agent.git
