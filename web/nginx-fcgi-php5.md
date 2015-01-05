Nginx + FastCGI + PHP5
======================

___WIP___

いわゆる LEMP 環境構築まで, Ubuntu(v14)で行う


## Nginx

### インストール
(詳細は略, apt-get OR src)

```
$ apt-get install nginx
```

### 設定

#### めも

```
$ ls -l /etc/nginx/
drwxr-xr-x 2 root root 4096  9月 17 22:47 conf.d
-rw-r--r-- 1 root root  911  3月  5  2014 fastcgi_params
-rw-r--r-- 1 root root 2258  3月  5  2014 koi-utf
-rw-r--r-- 1 root root 1805  3月  5  2014 koi-win
-rw-r--r-- 1 root root 2085  3月  5  2014 mime.types
-rw-r--r-- 1 root root  222  3月  5  2014 naxsi-ui.conf.1.4.1
-rw-r--r-- 1 root root  287  3月  5  2014 naxsi.rules
-rw-r--r-- 1 root root 5287  3月  5  2014 naxsi_core.rules
-rw-r--r-- 1 root root 1601  3月  5  2014 nginx.conf
-rw-r--r-- 1 root root  180  3月  5  2014 proxy_params
-rw-r--r-- 1 root root  465  3月  5  2014 scgi_params
drwxr-xr-x 2 root root 4096 12月 19 11:47 sites-available
drwxr-xr-x 2 root root 4096 12月 19 11:47 sites-enabled
-rw-r--r-- 1 root root  532  3月  5  2014 uwsgi_params
-rw-r--r-- 1 root root 3071  3月  5  2014 win-utf
```

* nginxの設定ファイルはcontextと呼ばれるブロックの階層構造になっており、より大きいブロックの設定値がデフォルト値として下方のブロックに伝わる

```
http  # nginxの全体的な設定
server  # バーチャルホストやlistenポート
location  # リクエストURLのパスをどう扱うか
```

* serverブロックはバーチャルホスト別に定義する
* apache同様、`conf.d`ディレクトリに存在するconfのincludeが __httpブロックに__ デフォルトで定義されている


#### 起動パラメータ系
* 設定すべき項目候補
    * `user`
    * `worker_processes`
        * [worker_processes](http://wiki.nginx.org/CoreModule#worker_processes)
    * `pid`

* パッケージインストール時のデフォルト

```nginx
user www-data;
worker_processes 4;
pid /run/nginx.pid;
```

#### イベント
* 設定すべき項目候補
    * `events`

* パッケージインストール時のデフォルト


```nginx
events {
    worker_connections 768;
    # multi_accept on;
}
```

#### 基本設定 = ポート, サーバー名, ドキュメントルート
* 設定すべき項目候補
    * `server`
        * `listen`
        * `server_name`
            * `localhost`でアクセスさせたい場合、`server_name _;`と`_`で省略する形式にする
        * `root`

        ```nginx
        listen       80  default_server
        listen       80  default_server

        server_name  example.org  www.example.org;
        ```

* サーバ名未定義(Hostヘッダが未定義)のリクエストを処理させたくない

```nginx
server {
    listen       80  default_server;
    server_name  _;  # 存在しないドメイン名 “_” をサーバ名として選択し
    return       444;  # 接続を閉じる nginx の特別な標準外コード 444 を返します
}
```


#### バーチャルホスト


#### インデックスページ
* 設定すべき項目候補
    * `location`
        * `index`


#### ログ
* 設定すべき項目候補
    * `server`
        * `access_log`
        * `error_log`


#### データ(レスポンス)圧縮
* 設定すべき項目候補
    * `server`
        * `gzip` - `gzip on`
        * `gzip_http_version`
        * `gzip_types`
        * `gzip_disable`
        * `gzip_vary`
        * `gzip_buffers`
        * `gzip_min_length`


#### アクセス制限
* 設定すべき項目候補
    * `location`
        * `allow`
        * `deny`

#### エラーページ
* 設定すべき項目候補
    * `server`
        * `error_page`

        ```nginx
        error_page 500 502 503 504 /50x.html;
        ```

#### リバースプロキシ
* 設定すべき項目候補
    * `location`
        * `proxy_pass`


### コマンド
* `nginx -t` - 設定ファイルのsyntaxチェック

### confのホットリロード
がそもそも可能なんだっけ？


### 参考
* [ConfigurationJa - Nginx Community](http://wiki.nginx.org/ConfigurationJa)
* [入門！ nginx - tumblr](http://shim0mura.hatenadiary.jp/entry/20120110/1326198429)
* [nginx はどのようにリクエストを処理するか](http://nginx.org/ja/docs/http/request_processing.html)
* [nginx連載4回目: nginxの設定、その2 - バーチャルサーバの設定 - インフラエンジニアway - Powered by HEARTBEATS](http://heartbeats.jp/hbblog/2012/04/nginx04.html)

## FastCGI

### インストール
(詳細は略, apt-get OR src)

```
$ apt-get install spawn-fcgi
```

### 設定


## PHP5 (5.5)

### インストール
* 最低限必要なパッケージ（が何か？）だけをまず入れる
    * `php5 php5-cgi php5-cli` or `php5-fpm` ??
        * FPM = FastCGI Process Manager

```
$ apt-get install php5-fpm
```

### 設定(php5自体)


### 設定(nginx + fastcgi連携)
* 設定すべき項目候補
    * `worker_processes`
    * `keepalive_timeout`

* 例

```nginx
location ~ \.php$ {
  include /etc/nginx/fcgi_params;
  fastcgi_pass  127.0.0.1:9000;
}
```

#### fastcgi
nginx.conf

* __"FCGIに関するすべての設定は1つのファイルにまとめた上でそのファイルをインポートするようにお勧めします"__
* 設定すべき項目候補
    * `location`
        * `include`
        * `fastcgi_pass`
        * `fastcgi_index`
        * `fastcgi_param`

#### キャッシュ


#### rewrite
* `.php`と拡張子剥き出しURLでアクセス受けるのはアレ ってのをnginx+phpの場合どうするのがベターか？
    * 使用するphpフレームワークのルーティング機能にも依存する？


### initスクリプト


### 参考
* [How To Install Linux, nginx, MySQL, PHP (LEMP) stack on Ubuntu 14.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04)
* [Nginx and PHP-FastCGI on Ubuntu 12.04 LTS (Precise Pangolin) - Linode Guides & Tutorials](https://www.linode.com/docs/websites/nginx/nginx-and-phpfastcgi-on-ubuntu-12-04-lts-precise-pangolin)
* [Setup Nginx + php-FPM + apc + MariaDB on Debian 7 – The perfect LEMP server](http://www.binarytides.com/install-nginx-php-fpm-mariadb-debian/)
* [Installing Nginx With PHP5 (And PHP-FPM) And MySQL Support (LEMP) On Ubuntu 14.04 LTS | HowtoForge - Linux Howtos and Tutorials](http://www.howtoforge.com/installing-nginx-with-php5-fpm-and-mysql-on-ubuntu-14.04-lts-lemp)
