### 下準備

#### boxの作成
- boxとは？
  - 仮想環境を構築するには、元となるOSが必要になります。本来は、ISOと呼ばれるOSのイメージfileを元に仮想環境を構築します。しかしvagrantの場合は、boxという形でOSのテンプレートfileを作成し、それを元に仮想環境をコマンドで作成することができます。なのでテンプレートとなるboxは、使い回しが聞くテンプレートとなります。
  - boxは、ISOと呼ばれるfileなどが集まったものとなってます
  
- 作成
  - boxを作成する際は、どこのディレクトリでも可能です
  ```shell
  vagrant box add BoxName URL
  ```
  - 上記のようなコマンドを打つと作成が行われます。
  - 例として以下のURLで作成したboxを元に進めていきます。
  ```shell
  vagrant box add vagrant_test https://github.com/holms/vagrant-centos7-box/releases/download/7.1.1503.001/CentOS-7.1.1503-x86_64-netboot.box
  ```
  - 上記コマンドを実行するとboxの作成が開始します
  

- Vagrantの作業ディレクトリを用意する
  - 場所は、任意で問題ありません
  - 今回のフォルダ名は、*vagrant_test*というフォルダ名にしましょう
  - boxの作成が終わったことを確認し作成したフォルダの中で以下のコマンドを実行します
  ```shell
  vagrant init vagrant_test
  # vagrant init BoxName とすることで先ほど作成したboxを使用することになります
  # 実行後問題なければ以下のような文言が表示されます
  ```

- Vagrant fileの編集
  - 今回行う編集は、二箇所です
    - *config.vm.network "forwarded_port", guest: 80, host: 8080*
    - *config.vm.network "private_network", ip: "192.168.33.10"*
  - 上記二箇所の # が付いているのを外します


- Vagrantの起動
  - ここまでの作業で仮想環境を構築する準備は整いました
  - 早速起動しましょう
  ```shell
  # Vagrantfileがあるディレクトリにて以下コマンドの実行
  vagrant up
  ```
  - 初回の起動には、そこそこ時間がかかりますのでコーヒーブレイクでもしましょう
  

- Vagrantの起動が終わったら
  - 起動が完了したら、仮想環境にアクセスしましょう
  - 簡単に説明しますとネットワークを通じてMacのターミナルのように操作することをSSH接続と言います
    - secure shellの略です
  - 今回はそのSSH接続を行います
  - まずは、MacでのSSH接続方法
  ```shell
  vagrant ssh
  
  # コマンドを実行した後以下のような表記になっていれば成功です
  Welcome to your Vagrant-built virtual machine.
  [vagrant@localhost ~]$
  ```
  - 次にWindowsでの接続方法
  - ターミナルエミュレーターを使用します
   - RLoginなどお好きなものを使用してください。こちらは、RLoginで作業を進めます
  - ip: 192.168.33.10
  - user: vagrant / password: vagrant
  - protocol: SSH
  - 上記を入力後接続が可能になるかと思います
  - 接続が行えたらMac同様の文言が表示されるかと思います
  - これでSSH接続は、できた状態になります


