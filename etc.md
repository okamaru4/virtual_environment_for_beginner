## 手順

- vagrant
- docker (docker-compose、laradock)

### Vagrant
  - 以下要件
    - かならずprivate ipを振っていること
    - 作業するディレクトリは、必ず/vagrant/ 以下であること
      - 表示する(Laravel)applicationがここに存在すること
    - かならずipアドレスでホストOS側からブラウザでアクセスできること
    - ゲストOS側のディレクトリに対してパーミッションの変更をしていないこと
    - firewall を止めていないこと

  - vagrant box add NAME URL
  - vagrant init NAME
  - Vagrantfileを編集します (private_network、portのコメントアウトを解除)
  - vagrant up
  - vagrant ssh
    - localhostに接続後
    - sudo yum -y groupinstall "development tools" // 開発に必要なライブラリをinstall

  - php install手順
```shell
sudo yum -y install epel-release
sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
sudo rpm -Uvh remi-release-7.rpm
sudo yum -y install --enablerepo=remi-php71 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel
```

  - Apacheを使用しての環境構築
    - sudo yum install -y httpd // apache
    - sudo firewall-cmd --add-service=http --zone=public --permanet
    - sudo firewall-cmd --reload
    - sudo systemctl start httpd
    - LAMP環境に必要なものを用意していく(mysql php)
      - 最新のLaravelだとphp70以上が必須となる
      - mysqlは5.7以上だとエラーなく使用することが可能です。5.7以下ですと手を加える必要があります

    - 問題点・つまずくポイント
      - apache：DocumentRootの場所 => こちらが作成したapplicationの場所を指定することを忘れがち
      - apache：実行者(起動するUser)の変更 => これを変更しないためLaravelの画面が線維できない
      - fierwallのhttp portの解放 => しないためブラウザで表示されない => 原因がわからないという状態になる
    - チェック方法
      - 実際にブラウザで表示してもらう
      - 実際に仮想環境にアクセスし、histroyコマンドを使用しコマンドの履歴をおい確認
      - それぞれ要件指定にあるディレクトリに対しチェックを行う

  - nginx + PHP-FPMを使用しての環境構築
    - 以下要件
    - apacheは使用しない
    - php-fpmを使用すること
    - nginxを使用すること
    - なおそれぞれ最新版を(できる限り)installすること
    - firewallを止めてないこと 

    - nginxのinstall方法：最新version

```shell
sudo vi /etc/yum.repo.d/nginx.repo

[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1

sudo yum install -y nginx
```
    - etc/nginx/conf.d/defalt.confを編集
    - 肝になる点
      - server_nameの変更
      - location内に記載あるrootをserver_name以下に記述
      - デフォルトのindex index.html index.htmと書かれている箇所にindex.phpを追記
      - location ~ .php$ { と書かれている箇所のroot以外コメントアウトを外す
      - その際必ず変更すべき箇所がfastcgi_paramの箇所 /scriptsとなっているのを /$document_root/と変更する
    - etc/php-fpm.d/www.conf
      - 変更箇所は、user = apache、group = apacheをnginxに変更する
```shell
sudo systemctl start nginx
sudo systemctl start php-fpm
```


- mysqlの導入
```shell
sudo rpm -ivh http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
sudo yum install -y mysql-community-server
sudo systemctl start mysqld
mysql -u root -p
PASSWORD: Enter
sudo cat /var/log/mysqld.log | grep 'temporary password'
2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: H1a0wiCfF&r&
# 表示されたpasswordを使用します H1a0wiCfF&r&の部分です
mysql -u root -p
# 接続が完了したらPasswordの設定を行います
# 本来ですとちゃんとした工程を踏むべきですが今回は、あくまで開発環境構築に慣れるということが目的なのでサクッと終わらせます
mysql > set password = "ここに新たなpasswordを入れます";
# passwordに関しては、ルールがあるので 英数字の大小文字+記号です
# 例として  Test_1234  このようなpasswordを最低8文字以上で入力します
# 終わったらdatabaseの作成です
mysql > crate database vagrant_test;
Query OK......
# となったら問題ないです
# exitで接続を切ってください
```


### laradock
  - 以下要件
    - nginx 、 apacheで表示が行えること
    - 複数のアプリケーションを動かすことができること
    - workspace内にてlaravelのapplicatoinの作成を行っていること
    - またcloneしてきたLaravelのapplicationも動かせること

  - 適当にディレクトリを作成
  - ディレクトリ内にてgit clone laradockURL
  - 新にlaradockのディレクトリが作成さるからディレクトリを移動
  - 移動後
  - cp env-example .env
  - docker-compose up -d nginx worksapce mysql (-d はバックグラウンド実行をするためのオプション)
  - 次にlaravelの導入
    - dcoker-compose exec workspace bash
    - 上記コマンドを実行しworkspaceに接続を行う
    - 接続したらls -alでフォルダの確認
    - 内容の確認したら次に行うのはlaravelの作成
    - composer create-project laravel/laravel lesson_laravel
    - 完了したらlaravelが作成されていることを確認する
  - nginx/sites/以下のfileを編集(編集箇所は、rootのPATH)
  - 終わったらdoker-compose up -d --build nginx
    - 安パイなのは、docker-compose stop > docker-compose up  nginx workspace
    - 明示的に停止を行った後に起動を行う
    - docker-compose up -dでバックグラウンド起動を行うことができる(デーモン)
  - localhostとブラウザに入れればLaravelのスタート画面が表示される


  - apache2 でlaravelを動かす
  - 最初にlaradock以下にある apache2/sites/default.apache.confを編集する
    - DocumentRoot と Directryの箇所、server nameをかえます。
    - 変更内容
      - 先ほど作成したLaravelのApplication以下のpublicまでのPAHTに変更
      - server nameを適当にかえます
      - windows、macそれぞれのhosts fileを変更します(先ほど設定したserver nameに)
  - docker-compose build apache2
  - docker-compose up -d apache2
