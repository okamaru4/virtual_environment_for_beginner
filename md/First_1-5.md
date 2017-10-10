# DBの設定を行う

- 今回使用するDBがmysqlとなります。導入するversionは、5.7を導入します。

## 導入

- centos7系は、デフォルトで入っているDBがmariaDBとなってます。がこれは、mysqlと互換性があるDBとなるのであまり気にせず、mysqlに必要なものをまずは、揃えます。
- ただし、既存で入っている。mariaDB関連が影響する場合もありますで不安な人は、あらかじめ消した方がいいでしょう。

rpmに新たにリポジトリを追加し、Installを行います。

```shell
sudo rpm -ivh http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
sudo yum install -y mysql-community-server
mysql --version
```

versionの確認ができましたら完了です。
次にmysqlを起動し接続を行います。

```shell
sudo systemctl start mysqld
mysql -u root -p
Enter password:
```

Macを使用しMacにmysqlを導入した場合は、これで接続が可能かと思いますが今回は、できなかったと思います。
接続できなかったのは、defaultでpasswordが設定されてしまっているからです。
なのでそのpasswordを調べ、接続しpassswordの再設定を行います。
※ここで注意が必要です。今回は、簡易的にpasswordを変更しているのでセキュリティ的には、よろしくなく本番環境と呼ばれる環境でのこの方法は避けましょう。別途記載予定です。仮想環境を構築するという点でここは進めます。

```shell
sudo cat /var/log/mysqld.log | grep 'temporary password'  # このコマンドを実行したら下記のように表示されたらOKです
2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: H1a0wiCfF&r&
```

一番右端に存在する向き列な文字列がpasswordとなります。なのでこれを使用します。

```shell
mysql -u root -p
Enter password: 
mysql > 
```

問題なく接続できましたでしょうか？
次に接続した状態でpasswordの変更を行います。

```shell
mysql > set password = "新たなpassword";
```

新たなpasswordには、必ず大文字小文字の英数字 + 記号かつ8文字以上を指定しましょう。

これで導入と設定が完了となります。

