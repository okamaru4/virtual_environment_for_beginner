# Windowsで環境を整える

- 問題点
  - Docker for windowsは、windows 10 Proのみで使用可能
  - 以前からある*toolbox*では、バグが存在している

上記状況を考え、今回は、Vagrantで作成した環境の中で *Linux for docker* を使用し *Laradock* を使いたいと思います。

## Dockerの導入

すでにVagrantでの環境がある前提になります。環境がない方は、第1章の方を行ってください。

導入手順は、公式サイトにある。[*Docker CE for CentOS*](https://docs.docker.com/engine/installation/linux/docker-ce/centos/)を踏襲し進めたいと思います。

導入にあたり必要なものをInstallしてねということなので行います。
※docker関連のパッケージを入れてない前提で行っています。

```shell
sudo yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
```

次に新たに`yum`にリポジトリの追加を行います。

```shell
sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```

それぞれコマンドは、以下と同義です

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

次に公式の手順に沿うと何やら有効化など出て来ますが一旦ここでは飛ばします。

早速DockerのInstallを行います。

```shell
sudo yum install -y docker-ce
```
今回は、docker-ceをInstallします。
※ceとは、Docker Community EditionのCommunity Editionを意味します。


```shell
docker -v
```
versionの確認ができましたでしょうか？

## Docker-Composeを導入します

dockerとは、また別に導入をする必要があります。こちらも公式を踏襲し導入していきます。
[Docker Compose](https://docs.docker.com/compose/install/#install-compose)こちらの*Linux*編を行います。

ひとまず、curlを使用し指定ディレクトリにダウンロードを行い、パーミッションを変え今使用しているUserでも使用可能にします。
```shell
sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
docker-compose version 1.16.1, build 1719ceb
```
と最終的に表示されたら問題ありません。

次に一度ssh接続をやめ再度ssh接続します。
※shellの再読み込みでも実行が確認できなかったため手間ですが再度接続し直します。

再接続を行ったら以下コマンドを実行しましょう。

```shell
docker-compose build workspace nginx apache2 mysql php-fpm
```

必要なパッケージを指定しBuildを行ってあげます。

そうすることによって仮想環境構築を行うことが可能になります。

※docker のimageをpullしそれを元に仮想環境を構築します。それらの定義書がDockerfileとなります。
※Vagrantのboxを作成しそれを元に仮想環境を構築します。それらの定義書がVagrantfileとなります。 >> これに非常に似ています。

```shell
docker-compose up -d apache2 workspace
```
とコマンドを打ったあとにvagrant で設定したipをブラウザに入力してください(ホスト側)。
そうしますと`Index of /`という文言が表示されましたでしょうか？

apacheの起動がこれで確認できました。


## DocumentRootなどの設定を行い、Laravelのwelcome pageの表示を行う

まず最初にapacheを使用して表示を行いたいと思います。
編集を行う対象fileは、*laradock/apache2/sites/default.apache.conf* となります。

```apache
<VirtualHost *:80>
  ServerName laravel.dev
  # ↓ 以下に編集
  ServerName 192.168.33.10 # → 自身のvagrant で設定したipを記載してください
  DocumentRoot /var/www/
  # ↓ 以下に編集
  DocumentRoot /var/www/project_name/public # vagrantの流れで行っているなら project_name はlaravel_testになると思います
  Options Indexes FollowSymLinks

  <Directory "/var/www/">
  # ↓ 以下に編集
  <Directory "/var/www/project_name/public"> # 上記のDocumentRootと同様に変えてください
    AllowOverride All
    <IfVersion < 2.4>
      Allow from all
    </IfVersion>
    <IfVersion >= 2.4>
      Require all granted
    </IfVersion>
  </Directory>

</VirtualHost>
```

編集を行ったら反映をさせるために起動停止後再起動しましょう。

```shell
docker-compose stop apache2
docker-compose up -d apache2
```

終わったらブラウザでipを入力の上アクセスしましょう。

問題なく表示ができましたでしょうか？


## Nginxを使用しての表示を行う

nginxもapache同様にrootの編集を行います。
編集するfileは、 *laradock/nginx/sites/default.conf* です。

```nginx
server {

    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    server_name localhost;
    # ↓ 以下に変更
    server_name 192.168.33.10; # 自身がvagrantで設定したipにしてください
    root /var/www/project_name/public; # ここもapache同様にlaravel_testになるかと思います。
    index index.php index.html index.htm;

    location / {
         try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        try_files $uri /index.php =404;
        fastcgi_pass php-upstream;
        fastcgi_index index.php;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    location /.well-known/acme-challenge/ {
        root /var/www/letsencrypt/;
        log_not_found off;
    }
}
```

編集が終わり起動状態ならばapache同様起動と停止を行いましょう。
流れですとapacheの停止を行い起動を行います。
```shell
docker-compose stop apache2
docker-compose up -d nginx
```

では、ブラウザで確認しましょう。
問題なく表示ができたら完了です。


## DBの設定を行う

