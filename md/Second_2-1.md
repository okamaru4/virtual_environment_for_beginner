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
でversionの確認を行ってください。
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
```
完了したらディレクトリを上記のように移動しましょう。


## laradockにあるものを必要なものだけbuildを行いましょう

- 今回必要になるものは、以下です。
  - workspace
  - mysql
  - apache
  - nginx

```shell
docker-compose build workspace mysql apache2 nginx
```

コマンドを実行したら、時間がかかりますで休憩でもしましょう。


## 早速Apacheで起動しブラウザから確認できるか確認しましょう

