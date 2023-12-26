# elastic-kibana-pi
Raspberry Pi Ubuntu 22.04 用の ElasticSearch + Kibana プリセット

### Usage

- Raspberry Pi 4B 2GB Ubuntu 22.04 64bit

### Raspberry Pi への Docker CE, Docker Compose のインストール

インストール

```bash
. scripts/install_docker_to_ubuntu.sh
sudo gpasswd -a [ユーザ名] docker
exit
# この後再ログイン
```

Docker CE, Docker Compose の動作確認

```bash
$ docker --version
# Docker version 20.10.17, build 100c701

$ docker compose version
# Docker Compose version v2.8.0

$ docker run --rm hello-world
# Hello from Docker!
# This message shows that your installation appears to be working correctly.
```

### ELK Stack のセットアップ

インストール

```bash
git clone https://github.com/deviantony/docker-elk.git
```

起動

```bash
docker compose up -d # インストールに30分以上かかることがある
```

### 課題
重すぎて動かない。`free` コマンドを実行すると以下のようになっていて、メモリが足りないことがわかる。  


|  | total | used | free | shared | buff/cache | available |
| --- | --- | --- | --- | --- | --- | --- |
| Mem: | 1889924 | 1839836 | 77348 | 3164 | 91784 | 50088 |
| Swap: | 0 | 0 | 0 |

Dockerが悪いのかもしれないし、ElasticSearch側のメモリ使用量などのなんらかの設定が悪いのかもしれない。
設定を変えて再検証する必要がある。まずは macOS を使って同じ手順を実行してみて、どの程度メモリを使用しているか確認してみる。


### 参考
- [Qiita - Linuxサーバー環境のDockerインストールメモ](https://qiita.com/ohhara_shiojiri/items/486a54ad895d6bb3144e)
- [Qiita - DockerでElastic Stack 8.2環境の構築](https://qiita.com/ohhara_shiojiri/items/0b45fd000103b7345073)
