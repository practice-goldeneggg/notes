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
___php5-fpm入れるんならこれ不要かも___

### インストール
(詳細は略, apt-get OR src)

```
$ apt-get install spawn-fcgi
```

### 設定
(TODO, 不要かも)


## php5-fpm (5.5)

### インストール
* 最低限必要なパッケージ（が何か？）だけをまず入れる
    * `php5 php5-cgi php5-cli` or `php5-fpm` ??
        * FPM = FastCGI Process Manager

```
$ apt-get install php5 php5-fpm
```

### 設定
* `cgi.fix_pathinfo=1`はセキュアじゃないので0に

```
$ sudo vi /etc/php5/fpm/php.ini

cgi.fix_pathinfo=0

$ sudo /etc/init.d/php5-fpm restart
```


## nginx + php5-fpm連携
* 設定すべき項目候補
    * `fastcgi_XXX`
* __"FCGIに関するすべての設定は1つのファイルにまとめた上でそのファイルをインポートするようにお勧めします"__
* 連携を行う仮想ホスト設定を`/etc/nginx/sites-available/HOST_NAME.conf`に作成
    * document rootはnginxのデフォルトの下に1階層subdirを掘る = `/usr/share/nginx/html/HOST_NAME/`

```nginx
server {
    listen 80;

    root /usr/share/nginx/html/hogefpm.localdomain;
    index index.php index.html index.htm;

    server_name hogefpm.localdomain;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock; # 実行中のFastCGIプロセスをNginxに接続する, "IP:PORT"形式も可。nginxとphp-fpmを別サーバに分散させる場合とか(複数台指定可能？）。1台完結ならsockで良い
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

* 作成した仮想ホスト設定を`/etc/nginx/sites-enable`へシムリンク張る
    * `nginx.conf`で`include /etc/nginx/sites-enable/*;`してるから、 「直接いじるのは`sites-available`下、その後`sites-enable`へシムリンクを張る」という例を多く見るけど、これが流儀？

```
$ ln -s /etc/nginx/sites-available/HOST_NAME.conf /etc/nginx/sites-enable
```

* document root `/usr/share/nginx/html/hogefpm.localdomain` に`index.php`を置く

```php
<?php
  phpinfo();
```

### 動作確認 using virtualbox
* `local machine -> (port forward) -> vm running nginx + php5-fpm`という構成での例
* local machineのhostsに、nginxで設定したホスト名をマッピング

```
$ sudo vi /etc/hosts

<IP ADDRESSS> hogefpm.localdomain
```

* ブラウザで`http://hogefpm.localdomain/index.php`へアクセスしてphpinfoが閲覧できることを確認


### initスクリプト


### 参考
* [How To Install Linux, nginx, MySQL, PHP (LEMP) stack on Ubuntu 14.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04)
* [Nginx and PHP-FastCGI on Ubuntu 12.04 LTS (Precise Pangolin) - Linode Guides & Tutorials](https://www.linode.com/docs/websites/nginx/nginx-and-phpfastcgi-on-ubuntu-12-04-lts-precise-pangolin)
* [Setup Nginx + php-FPM + apc + MariaDB on Debian 7 – The perfect LEMP server](http://www.binarytides.com/install-nginx-php-fpm-mariadb-debian/)
* [Installing Nginx With PHP5 (And PHP-FPM) And MySQL Support (LEMP) On Ubuntu 14.04 LTS | HowtoForge - Linux Howtos and Tutorials](http://www.howtoforge.com/installing-nginx-with-php5-fpm-and-mysql-on-ubuntu-14.04-lts-lemp)
