# Docker (Mac編)

## コンテナ型仮想環境と呼ばれるもの

### 特徴として
- インフラ(サーバ環境)をソースコードで管理することが可能で複数人での開発に向いています

- Vagrantやvirtual boxで行う仮想環境とは異なり、起動が早く、また使用するtoolごとにコンテナを分けて管理することが可能になります
 
- 必要なものがまとまっている。Docker for Mac を使用します。

- 前提として*Homebrew*がinstallされていることを想定しています。

 
## 準備

上記で述べたようにDocker for Macというものを使用します。

ダウンロード方法は2種類ありますが今回は、*Homebrew* 経由でinstallしたいと思います。

1. ご使用のPCにDocker関連のパッケージ等があるかどうか確認しましょう

```shell
brew list 
```
上記コマンド結果で表示された物の中にDockerに関するものがある方は、必ず*uninstall*を行ってください。

2. 早速Installを行います

```shell
brew cask install docker
```

これだけで十分です。Install中にpasswordを聞かれると思いますが自身のPCのpasswordを入力しEnterを押してください。
完了したら

```shell
docker --version
```
versionの確認を行ってください。
問題なくversionが確認できたら完了です。



## 早速起動を行い、必要なものをGithubからCloneして来ます

起動方法も大きく分け2種類存在しますがここはエンジニアらしくコマンドで実行しましょう。

```shell
open -a "docker"
```
このコマンドを実行したらMacの上部に存在するmenuの箇所にクジラが現れ、かつdockerのwindowが表示されましたでしょうか？
表示されれば問題ありません。

次に今回必要とするLaravelの環境構築するにあたり便利な*laradock*というものを使用します。
[Laradock](https://github.com/laradock/laradock)
上記リンク先のClone用URLを取得しCloneを行います。
Cloneするために任意のディレクトリを作成します。今回は、ディレクトリ名に*docker_test*としましょう。
```shell
cd docker_test
git clone https://github.com/laradock/laradock.git
cd laradock
cp env-example .env
```
完了したらディレクトリを上記のように移動しましょう。


## laradockにあるものを必要なものだけbuildを行いましょう

- 今回必要になるものは、以下です。
  - workspace
  - mysql
  - apache
  - nginx
  - php-fpm

```shell
docker-compose build workspace mysql apache2 nginx php-fpm
```

コマンドを実行したら、時間がかかりますで休憩でもしましょう。

buildし終わったら次にfileの編集を行います。このままですと`docker-compose up workspace apache2 mysql` としたところで何もブラウザで確認できません。
なのでまず最初にapacheの設定fileを編集します。


## Apacheのfileを編集


編集は、前回学んだ`Vagrant` の`apacheのhttp.conf` の編集に似てます。

対象fileは、`laradock/apache2/sites/default.apache.conf` です。
```apache
<VirtualHost *:80>
  ServerName laradock.dev
  # ↓ 変更
  ServerName localhost
  DocumentRoot /var/www/
  # ↓ 変更
  DocumentRoot /var/www/project_name/public/
  Options Indexes FollowSymLinks

  <Directory "/var/www/">
  #↓ 以下に変更です
  <Directory "/var/www/project_name/public">
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

この編集を終えた段階で再度`build` を行います。`build` を行なったら次に`up` を行うのですがこの際できれば起動のLogを吐き出し続けると行ったことをせずにバックグラウンドで起動をし続けてもらいたいので`up -d` とします。コマンドの全文は以下です。

```sql
docker-compose up -d workspace apache2
```

これでバックグラウンドでの起動が行えます。
早速ブラウザにて確認を行いたいと思います。*http://localhost* と入力をしましょう。


問題なく画面が表示されたら次に`nginx` を使用しての画面の表示を行います。

