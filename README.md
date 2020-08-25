# Ubuntu に Docker, docker-compose をセットアップして、PostgreSQL サーバを複数台構築する

Ubuntu のバージョンは 18.04 (bionic)

vagrant で Linux 環境を作り ssh ログインし、そこで Docker を起動する

ホスト IP は `192.33.10`

※ 最初は Docker 上の Linux コンテナ内で行おうとしたが、コンテナ内での Docker の起動がうまくいかなかったため、vagrant で環境構築を行った。

```
$ vagrant up
$ vagrant ssh
# もしパスワードを尋ねられたら "vagrant" を入力
```

```
パッケージリストの更新
# apt update
インストールされてるパッケージの更新
# apt -y upgrade

sudo, vimのインストール
# apt install -y sudo vim

作業ユーザー作成
# adduser inu
作業ユーザーにroot権限を与える
# usermod -aG sudo inu
作業ユーザーにログイン
# su inu
```

```
必要パッケージのインストール
$ sudo apt install -y apt-transport-https ca-certificates curl software-properties-common gnupg2 wget

Docker公式のGPG鍵を追加
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

Dockerのaptリポジトリを追加
$ sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

Docker CEのインストール
$ sudo apt-get install docker-ce docker-ce-cli containerd.io -y

作業ユーザーでDockerがつかえるように
$ sudo usermod -aG docker inu

/etc/docker ディレクトリを作成
$ sudo mkdir /etc/docker

デーモンのセットアップ
$ sudo vim /etc/docker/daemon.json

/// daemon.json ///
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
//////////////////

docker.service.dを作成
$ sudo mkdir -p /etc/systemd/system/docker.service.d

デーモンの再起動
$ sudo systemctl daemon-reload

Dockerの再起動
$ sudo systemctl restart docker

docker-composeのインストール
$ sudo apt install -y docker-compose

PostgreSQLサーバーのコンテナを立ち上げるためのdocker-compose.ymlを書く。
$ vim docker-compose.yml
// ・・・

docker-composeの実行
$ sudo docker-compose up -d

pg, psqlのインストール
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
$ echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
$ sudo apt update
$ sudo apt -y install postgresql-12 postgresql-client-12
```

##### VagrantのUbuntu上からDBサーバーにアクセスする
```
→ 自身のホストは localhost 扱い

$ psql -p 5433 -h localhost postgres_project_1 -U postgres_project_1
```

### 外部からDBサーバーにアクセスする
```
→ 仮想OSのホストは 192.168.33.10 (Vagrantfileに記載)

$ psql -p 5433 -h 192.168.33.10 postgres_project_1 -U postgres_project_1
```


## TODO:

- 環境変数・ポートを一元管理できるようにしたい
- プロジェクトが増えても簡単に PostgreSQL サーバを作れるようなシェルを作る
  (`ports.txt` みたいなポート管理のファイルと、シェル叩いたらポート自動生成 + プロジェクト名に応じた DB 作成)
- コンテナ外部からの接続ができるように調査

memo:  
[Ubuntu 最低限抑えておきたい初期設定](https://qiita.com/kotarella1110/items/f638822d64a43824dfa4)  
[ConoHa VPS (ubuntu 18.04) 初期設定メモ](https://qiita.com/jqtype/items/126c33ea176f3ba506c3)
