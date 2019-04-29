---
layout: default
title: トップページ
---

# トップページです
## 環境構築
### 2019/4/30
#### fishをインストール
#### vscodeをインストール
#### git をインストール
#### Docker CEをインストール
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
        取得:7 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
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

#### Dockerでwebアプリを実行
以下のサイトの内容を実施。

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
        $ curl http://localhost:4000
        <h3>Hello World!</h3>⏎
        $ docker ps
        CONTAINER ID        IMAGE                COMMAND             CREATED              STATUS              PORTS                  NAMES
        f19dd02ae2b8        docker-hello-world   "python app.py"     About a minute ago   Up About a minute   0.0.0.0:4000->80/tcp   infallible_torvalds