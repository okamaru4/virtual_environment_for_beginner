### Vagrantとは？

Vagrant
---
構成内容を記述したFileを元に仮想環境構築から設定までを自動で行うことができる
最近では、多方面の環境をサポートしたりすることにより用途の幅を広げている

多くのIT企業で採用されており、動作するOSを問わず環境構築が可能な点まだまだ利用されていくことが予想される

導入・設定
---

- 各OSごとに配布されているのでそれをインストールしましょう[Vagrant公式](https://www.vagrantup.com/)

- インストールが終わったらVagarntコマンドが使用可能かどうか確認するため以下のコマンドを実行しましょう

```shell
# windows
> vagrant -v

# mac
$ vagrant -v
```

versionの確認ができたら問題なく使用が可能状態です


頻出コマンド一覧
---
| コマンド           | 実行内容                                | オプション                                       |
| :-:                | :-:                                     | :-:                                              |
| vagrant init       | 設定ファイルの生成                      | box名を指定し作成するのが基本                    |
| vagrant box add    | boxの追加                               | URLを記述する場合は、ネット上のboxを元に作成する |
| vagrant box remove | boxの削除                               | box名をオプションで付与する                      |
| vagrant up         | 起動                                    |                                                  |
| vagrant ssh        | 仮想環境にssh接続を行う                 |                                                  |
| vagrant halt       | 起動中の仮想環境を停止                  |                                                  |
| vagrant reload     | 起動中の仮想環境を再起動                |                                                  |
| vagrant destroy    | 対象仮想環境の削除(ただしboxは消えない) |                                                  |
| vagrant status     | vagarntの起動状態の確認                 |                                                  |


上記のコマンドは、最頻出コマンドになりますので手に馴染ませましょう。
