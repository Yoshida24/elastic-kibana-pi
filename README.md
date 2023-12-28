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

インストールと起動

```bash
docker-compose up setup
docker-compose up # インストールに30分以上かかることがある
# [+] Running 5/5er-elk_elk  Creating                                                                                   0.0s 
# ✔ Network docker-elk_elk                Created                                                                      0.0s 
# ✔ Volume "docker-elk_elasticsearch"     Created                                                                      0.0s 
# ✔ Container docker-elk-elasticsearch-1  Started                                                                      1.0s 
# ✔ Container docker-elk-kibana-1         Started                                                                      0.1s 
# ✔ Container docker-elk-logstash-1       Started                                                                      0.1s 
```

ElasticSearchの疎通確認

```bash
curl http://localhost:9200 -u elastic:changeme

{
  "name" : "elasticsearch",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "jqtrJqwDTL-55hA_hTiSCw",
  "version" : {
    "number" : "8.11.3",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "64cf052f3b56b1fd4449f5454cb88aca7e739d9a",
    "build_date" : "2023-12-08T11:33:53.634979452Z",
    "build_snapshot" : false,
    "lucene_version" : "9.8.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

Kibana の疎通確認

```bash
# 数分待つ
open http://127.0.0.1:5601
```

### 課題
そのまま実行すると重すぎて動かない。`free` コマンドを実行すると以下のようになっていて、メモリが足りないことがわかる。  


|  | total | used | free | shared | buff/cache | available |
| --- | --- | --- | --- | --- | --- | --- |
| Mem: | 1889924 | 1839836 | 77348 | 3164 | 91784 | 50088 |
| Swap: | 0 | 0 | 0 |

Dockerが悪いのかもしれないし、ElasticSearch側のメモリ使用量などのなんらかの設定が悪いのかもしれない。
設定を変えて再検証する必要がある。まずは macOS を使って同じ手順を実行してみて、どの程度メモリを使用しているか確認してみる。

→ サーバ起動後に調べた結果 `docker stats` は以下の結果になった。  
CONTAINER ID   NAME                         CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O     PIDS
ba539103007f   docker-elk-logstash-1        13.59%    544.5MiB / 11.68GiB   4.55%     2.85kB / 2.06kB   0B / 315kB    66
b744969ecb7a   docker-elk-kibana-1          1.06%     578.3MiB / 11.68GiB   4.84%     13.4MB / 10.9MB   0B / 4.1kB    12
e72f6c7c9f78   docker-elk-elasticsearch-1   10.71%    1.142GiB / 11.68GiB   9.78%     4.11MB / 12.1MB   0B / 99.3MB   171
  
合計が2GBを超えているのでなんとかしないと動かない。

### 解決策
`docker-compose.yml` の以下の環境変数を128mに更新する（本来は256までが最低値ではあるが、一旦これでも動くようだ。）

```
LS_JAVA_OPTS: -Xms128m -Xmx128m
```

更新のためには `docker-compose up -d --build` を実行する。

`docker stats` のメモリ使用量はelasticsearchでは大きく改善する。合計が1.5GB程度になるので、2GBモデルのラズパイでもギリギリ動作する可能性がある。

CONTAINER ID   NAME                         CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O     PIDS  
7e524ac92f1e   docker-elk-logstash-1        2.31%     413.9MiB / 11.68GiB   3.46%     5.35kB / 2.67kB   0B / 279kB    66  
994526a79409   docker-elk-kibana-1          0.78%     545.2MiB / 11.68GiB   4.56%     1.1MB / 518kB     0B / 4.1kB    12  
8a691caf9182   docker-elk-elasticsearch-1   275.09%   572.3MiB / 11.68GiB   4.79%     31.2kB / 23.1kB   0B / 1.34MB   11  
  
メモリに余裕がなさすぎるのでまともに使えるものにはならないかもしれない。
Docker Image を省メモリなものに変更する方法を検討することは docker-elk を使っている限りは難しい可能性がある。
`docker.elastic.co/elasticsearch/elasticsearch` に依存してしまっているので、Image をそもそも変更できないため。
根本的に2GBモデルで動かすのに無理があるので、4GBモデル以上あることは必須になるかも。

### ラズパイで動かしてみる
この設定をラズパイで試してみて、実際にうまくいくか確認する。  
→ docker-compose V2 の導入に苦戦。

### 参考
- [Qiita - Linuxサーバー環境のDockerインストールメモ](https://qiita.com/ohhara_shiojiri/items/486a54ad895d6bb3144e)
- [Qiita - DockerでElastic Stack 8.2環境の構築](https://qiita.com/ohhara_shiojiri/items/0b45fd000103b7345073)
- https://github.com/deviantony/docker-elk