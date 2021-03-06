---
layout: default
title: トップページ
---
# 目的
docker関連技術の理解と応用の検討

# 目標
- dockerを用いた開発環境を構築する[済]
- dockerを用いたアプリを動作させる[済]
- 上記で使ったdockerコマンドを学習する[済]
- dockerリポジトリとしてgitlabを使ってみる[済]
- docker-composeの使い方を学習する
- elasticsearchの使い方を学習する

# 環境構築
### ubuntu 18.04.2LTSをインストール
### fishをインストール
### vscodeをインストール
#### 拡張機能
- vscode-icons
- Docker
### git をインストール
### Docker CEをインストール
https://docs.docker.com/install/linux/docker-ce/ubuntu/

1. リポジトリのアップデートは実施済み
1. パッケージをインストール

        sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
1. GPGキーを追加

        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

1. stable版のリポジトリを追加。fish用に変数の記述を修正。

        sudo add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        (lsb_release -cs) \
        stable"

1. 取得に失敗。`lsb_relase`が展開されていないのが気になるが、どれか一つでもリポジトリがヒットすればよいと判断。

        ヒット:1 http://jp.archive.ubuntu.com/ubuntu bionic InRelease
        ヒット:2 http://jp.archive.ubuntu.com/ubuntu bionic-updates InRelease                                                           
        取得:3 http://jp.archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]                                                 
        ヒット:4 http://packages.microsoft.com/repos/vscode stable InRelease                                                            
        無視:5 https://download.docker.com/linux/ubuntu (lsb_release InRelease                                                       
        取得:6 https://download.docker.com/linux/ubuntu bionic InRelease [64.4 kB]                     
        取得:7 http://s ecurity.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
        エラー:8 https://download.docker.com/linux/ubuntu (lsb_release Release
        404  Not Found [IP: 13.249.146.94 443]
        取得:9 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages [6,046 B]
        パッケージリストを読み込んでいます... 完了
        E: リポジトリ https://download.docker.com/linux/ubuntu (lsb_release Release には Release ファイルがありません。
        N: このようなリポジトリから更新を安全に行うことができないので、デフォルトでは更新が無効になっています。
        N: リポジトリの作成とユーザ設定の詳細は、apt-secure(8) man ページを参照してください。

1. docker ceをインストール

        $ sudo apt-get install docker-ce docker-ce-cli containerd.io

1. インストールできたことを確認

        $ sudo docker run hello-world

        Unable to find image 'hello-world:latest' locally
        latest: Pulling from library/hello-world
        1b930d010525: Pull complete 
        Digest: sha256:92695bc579f31df7a63da6922075d0666e565ceccad16b59c3374d2cf4e8e50e
        Status: Downloaded newer image for hello-world:latest

        Hello from Docker!
        This message shows that your installation appears to be working correctly.
        (略)

1. 一般ユーザでも`docker`コマンドが実行できるよう設定。

        $ sudo groupadd docker
        $ sudo usermod -aG docker $USER

        コマンド実行後、再起動を実施。

        $ docker run hello-world

## Dockerでpython webアプリを実行
以下のサイトの内容を実施。ホストのubuntuではpythonのバージョンは3.6.7だが、docker上では3.6.8が動作することになる。python3.6.8上でflaskベースのwebアプリ(`app.py`)を動作させる。

https://qiita.com/ksh-fthr/items/e914ff213791b7150008

1. ディレクトリを作成

        $ mkdir docker-hello-world

1. 以下のファイルを作成

        $ ls
        Dockerfile  app.py  requirements.txt

1. Dockerイメージを作成

        $ docker build -t docker-hello-world .
        $ docker image ls
        REPOSITORY           TAG                 IMAGE ID            CREATED              SIZE
        docker-hello-world   latest              c7002cee2ec3        About a minute ago   933MB
        python               3.6                 5281251bf064        4 days ago           924MB

1. Dockerイメージの起動

        $ docker run -d -p 4000:80 docker-hello-world

1. 動作確認

        $ curl http://localhost:4000
        <h3>Hello World!</h3>⏎
        $ docker ps
        CONTAINER ID        IMAGE                COMMAND             CREATED              STATUS              PORTS                  NAMES
        f19dd02ae2b8        docker-hello-world   "python app.py"     About a minute ago   Up About a minute   0.0.0.0:4000->80/tcp   infallible_torvalds

# dockerの学習
## dockerコマンドの学習
- `image`:Dockerイメージを管理する。
    - `docker images`: すべてのDockerイメージの一覧(`docker image ls`と同等)
- `build`:DockerfileからDockerイメージを作成する。
- `ps`:Dockerコンテナを管理する。
- `run`:新しいDockerコンテナ上でコマンドを実行する。
- `stop`と`kill`:`stop`はコンテナ内のプロセスに対し`SIGTERM`を送り、しばらくして`SIGKILL`を送る。`kill`はいきなり`SIGKILL`を送る。
- `rm`:1つ以上のDockerコンテナを削除する。
- `rmi`:1つ以上のDockerイメージを削除する。
- `save`と`export`:`save`はdockerイメージをtarアーカイブとして出力する。`export`はdockerコンテナのファイルシステムをtarアーカイブとして出力する。入力する際はそれぞれ`load`と`import`が対応する。
- `commit`:コンテナに対する変更からイメージを作成する。
- `tag`:dockerイメージに対してタグ付けを行う。構成名をスラッシュ区切りで指定する。デフォルトのタグ名は`latest`。プライベートリポジトリに登録する際はホスト名やポートを指定する。デフォルトのホスト名は`registry-1.docker.io`。

## Dockerfileの記法
dockerイメージの作り方を宣言的に書いた手順書のようなもの。
https://docs.docker.com/engine/reference/builder/

- `FROM`:ベースイメージを指定する命令。管理された、大きすぎないイメージを選択すること。
- `RUN`:OSのコマンドを実行する命令。既存イメージ上でコマンドを実行し、実行した結果をコミット(新しいイメージを作ること。イメージの更新？）する。`apt-get`等のインストールコマンドを指定することが一般的。
- `CMD`:コンテナ起動時に実行するコマンドを指定する命令。
- `EXPOSE`: コンテナが接続用にリッスンするポートを指定する命令。アプリケーションが一般的に使う伝統的なポートを指定するべき。Webアプリなら80等。外部からアクセスする際は`docker run -p ホストのポート:コンテナのポート`という書式で紐付ける。
- `ENV`:環境変数を指定する命令。PostgreSQLの`PGDATA`のような、サービスが必要とする環境変数を指定する場合にも使える。
- `ADD`と`COPY`:一般的には`COPY`が望ましい。ローカルファイルをコンテナにコピーするという用途に明確化されているため。`ADD`はtarアーカイブ展開やリモートURLにも対応しており多機能だが、一見して処理内容がわかりにくい。`ADD`はローカルのtarアーカイブをコンテナ内に展開する際に限定すること。
- `WORKDIR`:Dockerfile内の命令実行時の作業ディレクトリを指定する命令。`WORKDIR`で設定したディレクトリが`docker run`時の作業ディレクトリになる。Dockerfileの記述がすっきりするため積極的に使うこと。

# Dockerリポジトリの作成
## Gitlabの導入、設定
1. www.gitlab.comにユーザ登録
1. 以下のプロジェクトを作成する
    - https://gitlab.com/u5ke/test
    - Private

## Gitlabへの登録
GitlabへDockerイメージを登録する。  
**`docker login`が成功せず。調査要**  
→解決した。githubの認証情報を利用してgitlabに登録していたが、`docker login`で使える認証情報ではなかったと思われる。account settingsでパスワードを登録したところ`docker login`でのログインに成功した。

1. Gitlabへログインする。

        $ docker login registry.gitlab.com
        Login Succeeded

1. Dockerイメージにタグ付けする

        $ docker images
        REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
        docker-hello-world        python-app          d5936d915e43        11 days ago         933MB
        python                    3.6                 5281251bf064        2 weeks ago         924MB
        $ docker tag docker-hello-world:python-app registry.gitlab.com/u5ke/test
        $ docker images
        REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
        docker-hello-world              python-app          d5936d915e43        11 days ago         933MB
        registry.gitlab.com/u5ke/test   latest              d5936d915e43        11 days ago         933MB
        python                          3.6                 5281251bf064        2 weeks ago         924MB

1. GitlabにDockerイメージをpushする。

        $ docker push registry.gitlab.com/u5ke/test
        The push refers to repository [registry.gitlab.com/u5ke/test]
        0a87f2caa24c: Pushed 
        8469a017122f: Pushed 
        d3dce31b2149: Pushed 
        5d5d86989c42: Pushed 
        8296c34d0641: Pushed 
        4d29b148a147: Pushed 
        bd5afbfde642: Pushed 
        0fe19df8b8f8: Pushed 
        b17cc31e431b: Pushed 
        12cb127eee44: Pushed 
        604829a174eb: Pushed 
        fbb641a8b943: Pushed 
        latest: digest: sha256:716ef1225a40ede07804c5b88898319c76b616554795a797a8565e73718a5756 size: 2843

### 参考
何個でもpushできる。

        $ docker tag python:3.6 registry.gitlab.com/u5ke/test/python:3.6
        $ docker images
        REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
        docker-hello-world                     python-app          d5936d915e43        11 days ago         933MB
        registry.gitlab.com/u5ke/test          latest              d5936d915e43        11 days ago         933MB
        python                                 3.6                 5281251bf064        2 weeks ago         924MB
        registry.gitlab.com/u5ke/test/python   3.6                 5281251bf064        2 weeks ago         924MB
        $ docker push registry.gitlab.com/u5ke/test/python:3.6
        The push refers to repository [registry.gitlab.com/u5ke/test/python]
        5d5d86989c42: Mounted from u5ke/test 
        8296c34d0641: Mounted from u5ke/test 
        4d29b148a147: Mounted from u5ke/test 
        bd5afbfde642: Mounted from u5ke/test 
        0fe19df8b8f8: Mounted from u5ke/test 
        b17cc31e431b: Mounted from u5ke/test 
        12cb127eee44: Mounted from u5ke/test 
        604829a174eb: Mounted from u5ke/test 
        fbb641a8b943: Mounted from u5ke/test 
        3.6: digest: sha256:128a8dc57c2cff9e4ff2e4e7a66612e179aff6e7043ae7f80f3ee55b8bb7baec size: 2218

## Gitlabからpullする

```fish
$ docker images
REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
docker-hello-world                     python-app          d5936d915e43        11 days ago         933MB
python                                 3.6                 5281251bf064        2 weeks ago         924MB
registry.gitlab.com/u5ke/test/python   3.6                 5281251bf064        2 weeks ago         924MB
$ docker pull registry.gitlab.com/u5ke/test
Using default tag: latest
latest: Pulling from u5ke/test
Digest: sha256:716ef1225a40ede07804c5b88898319c76b616554795a797a8565e73718a5756
Status: Downloaded newer image for registry.gitlab.com/u5ke/test:latest
$ docker images
REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
docker-hello-world                     python-app          d5936d915e43        11 days ago         933MB
registry.gitlab.com/u5ke/test          latest              d5936d915e43        11 days ago         933MB
python                                 3.6                 5281251bf064        2 weeks ago         924MB
registry.gitlab.com/u5ke/test/python   3.6                 5281251bf064        2 weeks ago         924MB
```
# elasticsearchの学習
https://blog.shibayu36.org/entry/2016/08/31/110000
## mapping
indexだけ作成して登録してもdynamic mappingによってmappingは作られるが、大半の場合は設計が必要になる。
### datatype `type`
- `text`: 全文検索対象。`analyzer`プロパティで形態素解析器等を設定する。ソートやaggregation(集約して最大値等を求める処理)には使用されない。significant text aggregationは除く。
- `keyword`：全文検索対象でない。`analyzer`は設定できない。メールアドレスやホスト名、タグのようにフィルタリングで使うような値に設定する。exact matchより柔軟にしたい場合は`normalizer`を使う。
- `date`：Elasticsearchにおいてはフォーマットされた日付を含む文字列、milliseconds-since-the-epochで表現されるlong、seconds-since-the-epochで表現されるintegerのこと。内部的にはUTCに変換されmilliseconds-since-the-epochで表現される`long`として格納される。
- `long`：
- `double`：
- `boolean`：

## bulk API 
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-bulk.html


# メモ
- javaのwebアプリのdockerイメージを作る場合、tomcatのdockerイメージを作り、webappsフォルダ配下に.warファイルを配置しておけばよさそう。ポートの設定はtomcatのserver.xmlとかspringのapplications.propertiesで？
    - 組み込みjettyやtomcatの場合はどこに配置してもよいのでそちらを推奨とするのもよさそう
    - javaアプリをdocker化するツール=jib。mavenで使える。
- dockerコンテナをオフライン環境に持っていく方法は。fatjarのようなコンテナの作り方。ベースイメージに`latest`が指定されているとビルドのたびに依存しているコンテナのバージョンが変わる危険性がある。一度ベースとしたものはオフラインに持って来てオフラインで管理する必要あり。`docker save`や`docker export`を活用？オフラインイメージの管理はgitlabがいいとのこと。
https://blog.nownabe.com/2018/02/17/1259.html
- dockerイメージの脆弱性検査？
- noderedのオリジナルノードはnpmモジュールで作る必要がある。
https://qiita.com/zuhito/items/1c65fdbb3743d9f87edd
- コンテナを停止しても、コンテナ内の領域にデータは残る。コンテナ＝マシンと同等。
- `docker run -v`でホストのディレクトリと紐付ける際は、ホストのディレクトリの権限に注意。`ll`、`chgrp`や`docker run --user`を併用して正しい権限を付与すること。
- kubernetesを使わない使い方も用意すべきではないか。必要な技術が多くなるほどハードルが上がってしまう。同様にdockerfileも必須ではない。まずは開発環境でdockerを使って慣れてもらう等が必要ではないか。VM環境との共存の余地を残すことも重要。ソフトランディングできないと使ってもらえない。
- 既存アプリをdocker化する手順を整理する必要あり。
    - 役割に応じたコンテナの分離。  
      Tomcat+JavaのWebアプリ+PostgreSQLのような構成だとコンテナは2つ(Tomcat+JavaのWebアプリと、PostgreSQL)になる。
    - コンテナ上で開発し、イメージを作成する。(`docker commit`やDockerfile)
    - エントリポイントを考える。  
      アプリコンテナとPostgreSQLコンテナを通信させるためには、PostgreSQLコンテナに`--name`オプションで名前をつけ、その名前をアプリコンテナから`--link`オプションで紐付けることで利用可能。
