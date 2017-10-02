### パッケージを導入します

- パッケージを導入するにあたり頻出となるコマンド
```shell
sudo yum -y install パッケージ名
sudo yum -y groupinstall "導入する名称"
```
  - sudo：Root(システム管理者以下Root)の権限を借りる
    - VagrantでSSH接続した際
    - 仮想環境にアクセスしてるのは、vagrantというUserでSSH接続を行っています
    - このUserは、Rootによって作成されたUserでシステムに対しての操作を行う際は、必ずRootの権限を一次的に借り操作を行います
    - 通常Rootでの何かしらの実行というのは行わないということは覚えておきましょう
  - yum
    - 今回仮想環境を構築しているのは、Linuxと呼ばれるOSの種類のCentOSを使用しています
    - yumとは、CentOSを代表とするLinuxOSのRedHat系ディストリビューションと呼ばれる種類のOSで使用されているパッケージ管理ツールです
      - Macでいうbrewなどをイメージしていただければいいかと思います
    - *-y*に付いて
      - こちらは、オプションと呼びinstallを行っている際に yes/noなどと聞かれるのを全てyesとするためのものです
      - 特に大量にパッケージをinstallする際は、つけたほうがinstallが最後まで手を動かすことなく可能となります
      - また今回は、vagrantの入門になるので全てに*-y*をいれていいと思います
  - groupinstall
    - 通常のinstallと異なり、パッケージをまとめて導入することが可能です
    - 今回この後実行する最初のコマンドで登場しますがそれ以外にも種類があるのでぜひ検索してみてください
    
- 環境構築前に必要なパッケージを導入
  - 最初に必要なものを一気に導入します
  ```shell
  sudo yum -y groupinstall "development tools"
  ```
  - このコマンドを実行することにより一気に開発に必要なパッケージをinstallができます

- 環境構築を始めます
  - 今回の最終目的がvagrantを使用してLaravel(PHP FW)のwelcome pageを表示するところまでです
  - 必要となるパッケージとして
    - apache / nginx (両方で表示ができるようにします)
    - php version 7.*.*
    - 現段階では、アプリケーションとして何か動作させるわけではないのでDBは、後ほど課題の際に導入してもらいます
  - ではinstallの手順です
  ```shell
  sudo yum -y install httpd
  ```
  - 上記コマンドでapacheのinstallが完了
  - 次にphpの最新をinstallします
  - apacheと同様なinstallですと古いphpがinstallされます。今回必要となるphpのversionが7以上です(Laravelの最新のversionは、phpのversionが7以上と指定されています)。古いversionをinstallするのでなく新しいversionをinstallするには、CentoOSに最新のphpが取得できるように設定を行ってあげます
  ```shell
  sudo yum -y install epel-release
  # yumがパッケージを取得しにいく場所の更新を行います
  sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
  sudo rpm -Uvh remi-release-7.rpm
  sudo yum -y install --enablerepo=remi-php71 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel
  php -v
  ```
  - phpのversionが確認できたでしょうか？ phpのinstallに関しては、以上で終わりです
  - apaceh + php を使用してのLaravelのwelcome pageの表示を行います
  - LaravelのProject作成には、composerが必要になりますのでcomposerのinstallを行います 
  - 公式のDownload手順を踏襲します
  ```shell
  php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
  php composer-setup.php
  php -r "unlink('composer-setup.php');"
  # グローバルにコマンドを使用するためにfileの移動を行います
  sudo mv composer.phar /usr/local/bin/composer
  composer -v
  ```
  - composer のversionが確認できたら完了です
  - この状態でLaravelのProjectを作成する準備は、終わりました
  

- Laravelを導入する
  - Laravelを作成するコマンドがありますでそれを実行します
  ```shell
  # まず最初にLaravelを作成するディレクトリに移動します
  cd /vagrant
  composer create-project laravel/laravel laravel_test
  ```
  - 上記コマンドを実行すると少し時間がかかりますがLaravelのProjectが作成されます
  
- apacheの設定fileにLaravelのProjectまでのPATHに変更する
  - apacheの設定fileは、*/etc/httpd/conf/httpd.conf* にありますのでこのfileをRootの権限を借りて編集します
  - 編集には、普段使うエディタを使用せず元々CentOSに存在する*vi*を使用し行います。
  ```shell
  # 必ずsudoを使用しなければvagrant userでは編集できません
  sudo vi /etc/httpd/conf/http.conf
  ```
  - 編集箇所は、大きく分けて3箇所あります
    - DocumentRoot
    ```
    DocumentRoot "/var/www/html" # ここを修正
    DocumentRoot "/vagrant/laravel_test/public"
    ```
    - Directory
    ```
    <Directory "/var/www/">
        AllowOverride None
        Require all granted
    </Directory>
    ↓  以下に変更
    <Directory "/vagrant/laravel_test/public"> # ここを変更
        AllowOverride All  # ここを変更
        Require all granted
    </Directory>
    ```
    - User, Group
    ```
    User apache
    Group apache
    ↓  以下に変更
    User vagrant
    Group vagrant
    ```
  - 編集が終わったら一度apacheを起動しましょう！
  ```shell
  sudo systemctl start httpd
  ```
  - 起動しましたらホストOS側でブラウザに *http://192.168.33.10* と入力し確認して見ましょう
  - ブラウザ画面には、Laravelのwelcomeページが表示されましたでしょうか？
    - おそらく答えは、「No」だと思います
  - 画面すら表示されずの状態かと思います
  - ではなぜこのような画面が表示されているのか、答えは、ブラウザに書いてあります。
  - 表示されている文言の下部に「ファイヤーウォール」という単語が確認できますでしょうか？聞きなれない単語かと思いますがセキュリティという観点では、「ファイヤーウォール」というのは、大切な機能です。なのでできればこの機能は、止めないように起動状態のままホストOS側からアクセスできるようにしたいのです
    - Vagrantfileの編集をした際を思い出してください。コメントアウトされてた箇所を二箇所有効にしました
    - その有効にした箇所でconfig.vm.network "forwarded_port", guest: 80, host: 8080の記述がありました。この80というのは、httpという通信を行うためのポートと呼ばれる窓口です
    - なのでファイヤーウォールに対してこのポートを経由したhttp通信によるアクセスを許可するためのコマンドを実行します
  ```shell
  sudo firewall-cmd --add-service=http --zone=public --permanet
  # 新たに追加を行ったのでそれをファイヤーウォールに反映させるコマンドも合わせて実行します
  sudo firewall-cmd --reload
  ```
  - では、一旦この状態で画面を確認しましょう
  
