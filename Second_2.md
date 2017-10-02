### 概要説明と頻出コマンド一覧

- Dockerとは？
  - 仮想環境構築を行うためのToolです
  - 今までものとは異なり、起動などが迅速であるという点が挙げられます
  - 細かくは割愛します

- Docker Composeとは？
  - Dockerは、一個のコンテナでも開発環境を構築することは可能ですが、できれば一個のコンテナでは、一個のプロセスのみにしたい、後々流量のしたい、一個のコンテナの大きさを小さく保ちたいといったことは、安易に想定できます。またそうでなかったとしても複数のコンテナをコンテナ単位で起動せざる負えない状況になってしまうのは、とても助長で手数が増えます。そこでそれを解消するためのものが *Docker Compose* です
  - Docker Composeを使用してdocker-compose.ymlというもので複数のコンテナを一括起動、停止を可能とすることができます 
  - Laradockとは？
    - Laradockは、Laravelの開発環境をDocker Composeを使用して簡単に構築ができるものです

- 頻出コマンド ~ Laradcokを使用する上で ~ 
```shell
docker-compose build # Dockerfileを元にイメージを取得後コンテナの作成を行う
docker-compose up    # コンテナの起動
docker-compose ps    # コンテナのプロセスの確認 起動中ならUp / 停止状態ならExit
docker-compose up -d # コンテナをデーモン(バックエンド)で起動
```

