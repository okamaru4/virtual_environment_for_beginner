### 次にNginxを利用しての画面の表示を行います

- 前回作成した*Laravel*を使用します

- 目的としてapacheの次にシェアを拡大している*Nginx*を使用しての*Laravel*のwelcome画面の表示です
  - まず起動状態であるapacheを一度停止しましょう
  ```shell
  sudo systemctl stop httpd
  ```
  - これで停止出来たかなと思います
  - nginxを使用する上で必要となるものが*php-fpm*です
    - こちら先ほどのphpの導入の段階でInstallを済ませてます


- NginxのInstall
  - できるかぎり最新をInstallしたいと思います
  ```shell
  sudo vi /etc/yum.repos.d/nginx.repo
  ```
  - 上記コマンドを作成したら自動で*vi*というエディタが立ち上がりますので下記内容を書きましょう
  ```
  [nginx]
  name=nginx repo
  baseurl=http://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
  gpgcheck=0
  enabled=1
  ```
  - 書き終わったら*esc + :wq*を実行しましょう
    - viというエディタの操作方法は、検索するとすぐ出てきますがこちらの方でも最後の方で基礎的なものはまとめたいと思います
  - fileの保存が出来ましたら以下のコマンドを実行しNginxのInstallを行います
  ```shell
  sudo yum install -y nginx
  nginx -v
  ```
  - nginxのversionが確認できたでしょうか？
  - 確認ができたら問題ありません。この状態でブラウザでnginxの動作確認を行いたいので一度起動しましょう
  ```shell
  sudo systemctl start nginx
  ```
  - 次にブラウザにて *192.168.33.10* と入力しwelcomeページが表示されましたでしょうか？
    - 表示されたら問題なく動いていることが確認できます
    

- Nginxを使用して*Laravel*の画面を表示
  - apacheでLaravelを表示した際と同様に設定fileがnginxにも存在しているので編集を行います
  - それに伴いapacehと異なりnginxに関しては、*php-fpm*という物を使用すると話をしました
    - このライブラリにも設定fileが存在しているのでこちらも編集を行います
  
  - では、早速nginxの設定fileを編集していきます
  ```shell
  sudo vi /etc/nginx/conf.d/default.conf
  ```
  - 編集範囲が広いですが頑張りましょう。
  
  ```nginx
  server {
    listen       80;
    server_name  192.168.33.10; # ここのipに関しては、自身の環境に合わせてください
    root /vagrant/laravel_test/public; # 追記
    index  index.html index.htm index.php; # 追記


    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        #root   /usr/share/nginx/html; # コメントアウト
        #index  index.html index.htm;  # コメントアウト
        try_files $uri $uri/ /index.php$is_args$args;  # 追記
    }
    
    # 省略
    
    # 以下の該当箇所のコメントアウトを指定の箇所外し、変更する場所もあるので変更を加える
    location ~ \.php$ {
    #    root           html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
        include        fastcgi_params;
    }
    
    # 省略
  ```
  
  - nginxの設定fileの変更は、以上です
  
  - 次にnginxを使用する上で必ず必要になる*php-fpm*の設定fileを編集します
  ```shell
  sudo vi /etc/php-fpm.d/www.conf
  ```
  
  - 変更箇所は以下になります
  
   ```php-fpm
   ;24行目近辺
   user = apache
   ↓ 変更
   user = nginx
   group = apache
   ↓ 変更
   group = nginx
   ```
   
   - 設定fileの変更に関しては、以上となります
   
   - 早速起動しましょう(nginxは再起動になります)
   ```shell
   sudo systemctl restart nginx
   sudo systemctl start php-fpm
   ```
   
   - では、ブラウザにて確認しましょう
   
