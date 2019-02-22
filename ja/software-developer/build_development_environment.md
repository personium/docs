# 開発用環境の構築手順

## 対象OS
Windows

## 開発に必要なツール一覧
開発に必要なツールをダウンロードしてください。

| ツール名 | バージョン | ダウンロードURL |
|:--|:--|:--|
| Eclipse (Pleiades) | 2018-09 Full Edition 以降 | http://mergedoc.osdn.jp/ |
| OracleJDK | 1.8系 | Pleiades同梱のものを使用するため個別の導入は行わない |
| Git | 最新 | https://git-scm.com/ |
| Maven | 3系 | https://maven.apache.org/download.cgi |

## 動作確認に必要なツール一覧
動作確認に必要なツールをダウンロードしてください。

| ツール名 | バージョン | ダウンロードURL |
|:--|:--|:--|
| Tomcat | 9.0系 | Pleiades同梱のものを使用するため個別の導入は行わない |
| Elasticsearch | 5.6.14 | https://www.elastic.co/jp/downloads/past-releases/elasticsearch-5-6-14 |
| ActiveMQ | 5.15系 | http://activemq.apache.org/download-archives.html |
| Nginx | 1.14系 | http://nginx.org/download/nginx-1.14.0.zip |


## 各種ツールの初期設定
コマンドは管理者権限を持つユーザで実行してください。
### OracleJDK
環境変数(JAVA_HOME)を設定します。  

```
変数：JAVA_HOME
値：javaのインストールディレクトリパス(例：C:\Program Files\pleiades\java\8)
```

システム環境変数Pathに以下を追加します。

```
%JAVA_HOME%\bin
```

### Git
C:¥Users¥[user]¥.gitconfigに、以下の記述を追加します。  

```
[core]
	autocrlf = false
 	ignorecase = false
```

> プロキシ環境下に開発環境を構築する場合、以下も追記します。  
※{}の値はユーザ自身の情報を入力します。

```
[http]
	proxy = {protocol}://{username}:{pass}@{host}:{port}
[https]
	proxy = {protocol}://{username}:{pass}@{host}:{port}
```


### Maven

環境変数(M2_HOME)を設定します。  

```
変数：M2_HOME
値：mavenのインストールディレクトリパス（例：C:\Program Files\apache-maven-3.5.4）
```

システム環境変数Pathに以下を追加します。

```
%M2_HOME%\bin
```

> プロキシ環境下に開発環境を構築する場合、apache-maven-3.5.4\conf\settings.xmlに以下の記述を追加します。  
※{}の値はユーザ自身の情報を入力します。

```
<proxies>
  <proxy>
      <id>proxy00</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>{username}</username>
      <password>{pass}</password>
      <host>{host}</host>
      <port>{port}</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
  </proxy>
  <proxy>
      <id>proxy01</id>
      <active>true</active>
      <protocol>https</protocol>
      <username>{username}</username>
      <password>{pass}</password>
      <host>{host}</host>
      <port>{port}</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
  </proxy>
</proxies>
```

### Elasticsearch
#### elasticsearch-5.6.14\config\elasticsearch.ymlの修正

elasticsearch.ymlに以下の記述を追加します。

```
cluster.name: 【任意の名前】

index.number_of_shards: 1
index.number_of_replicas: 0
index.max_result_window: 200000
index.merge.scheduler.max_thread_count: 1
action.auto_create_index: false
index.refresh_interval : -1
http.cors.enabled: true
http.cors.allow-origin: "*"
indices.fielddata.cache.size: 80%（任意:ヒープメモリの何％をデータキャッシュとして使用するか）
```

#### Elasticsearch用のPluginの導入
##### delete-by-query Plugin  
セルの再帰削除API時に使用するためのPluginを導入します。  
こちらのPluginは導入必須となります。  

コマンドプロンプト等でElasticsearchのインストールディレクトリに移動します。

```
> cd Elasticsearchインストールディレクトリ
```

以下コマンドを実行します。

```
> bin\plugin install delete-by-query
```

##### elasticsearch-head Plugin  
ElasticSearchに保存されているデータを見やすくするためのPluginを導入します。  
こちらのPluginは導入推奨となります。  

コマンドプロンプト等でElasticsearchのインストールディレクトリに移動します。

```
> cd Elasticsearchインストールディレクトリ
```

以下コマンドを実行します。

```
> bin\plugin install mobz/elasticsearch-head
```

> プロキシ環境下でPluginの導入がうまくできない場合、  
以下に記載した別途ダウンロードしてきたPluginをローカルで導入する手順を実施してください。  

1. Plugin導入のコマンドを実行します。  
失敗した場合、以下のメッセージが出力されます。

    ```
    > bin\plugin install mobz/elasticsearch-head
    -> Installing mobz/elasticsearch-head...
    Trying https://github.com/mobz/elasticsearch-head/archive/master.zip ...
    ERROR: failed to download out of all possible locations..., use --verbose to get detailed information
    ```

1. 「Trying 」から「 ...」までに出力されたURLでPlugin本体をダウンロード可能となります。  
ブラウザなどでダウンロードし、適当な場所に置きます。  
C:\Tools\elasticsearch-5.6.14\tmp\elasticsearch-head-master.zip

1. Plugin installコマンドをURI指定で実行します。

    ```
    > bin\plugin install file:C:\Tools\elasticsearch-5.6.14\tmp\elasticsearch-head-master.zip
    ```

### ActiveMQ
特になし


### Tomcat
特になし


### Nginx
Nginxの初期設定のため、nginx-1.14.0\conf配下のnginx.confファイルを修正します。  

1. httpの中に以下記述を追加します。

    ```
        ignore_invalid_headers on;
        proxy_set_header Host $http_host;
        proxy_http_version 1.1;
        large_client_header_buffers 4 55k;

        map $http_upgrade $connection_upgrade {
          default upgrade;
          '' close;
        }
    ```

1. http.server.location /の内容を以下記述に変更します。

    ```
                #root   html;
                #index  index.html index.htm;

                if ($request_uri ~ [\x00-\x20\x22\x3c\x3e\x5b-\x5e\x60\x7b-\x7d\x7f]) {
                    return 400;
                }

                if ($request_uri ~* ([^?]+)\?(.*)) {
                  set $personium_path $1;
                  rewrite .* /personium-core$personium_path break;
                }
                if ($is_args = "") {
                  rewrite .* /personium-core$request_uri break;
                }

                proxy_pass http://localhost:8080;
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_set_header X-Forwarded-Proto http;
                proxy_set_header X-Forwarded-Path $request_uri;
                proxy_set_header Host $http_host;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                proxy_hide_header X-Powered-By;
                proxy_hide_header X-Rack-Cache;
                proxy_hide_header X-Content-Digest;
                proxy_hide_header X-Runtime;
    ```

## Eclipseへのプロジェクト登録
personium-coreとpersonium-engineをプロジェクトとして登録します。  
1. GitHubのリポジトリからcloneします。  
https://github.com/personium/personium-core.git  
https://github.com/personium/personium-engine.git

1. Eclipseのmavenプロジェクトとしてそれぞれを登録します。  

1. それぞれの/src/main/resourcesにpersonium-unit-config.propertiesを作成します。
> io.personium.core.es.cluster.nameの値はElasticsearchのconfigファイル（elasticsearch.yml）  
のcluster.nameで設定した値を指定します。

**/personium_core/src/main/resources/personium-unit-config.properties**

```
#################################################
# personium-core configurations
#################################################
# general configurations
io.personium.core.masterToken=personiumio
io.personium.core.unitScheme=http
#io.personium.core.unitPort=9998
#io.personium.core.unitPort=8080
io.personium.core.unitPort=
#io.personium.core.unitPath=/personium-core
io.personium.core.unitPath=

io.personium.core.unitUser.issuers=personium-localunit:/unitadmin/ personium-localunit:/unitadmincell/ personium-localunit:/unitusercell/

# cell configurations
io.personium.core.cell.relayhtmlurl.default=https://demo.personium.io/app-cc-home/__/index.html

io.personium.core.es.cluster.name=【elasticsearch.ymlで設定した値】
io.personium.core.es.retryTimes=10

io.personium.core.plugin.path=/personium/plugins
io.personium.core.blobStore.root=/personium_nfs/personium-core/dav
io.personium.core.bar.tmp.dir=/personium_nfs/personium-core/bar
io.personium.core.event.log.current.dir=/personium_nfs/personium-core/eventlog
io.personium.core.cellSnapshot.root=/personium_nfs/personium-core/snapshot

# lock type configurations
io.personium.core.lock.type=inProcess

# cache configurations (memcached protocol)
io.personium.core.cache.type=inProcess
io.personium.core.cache.cell.enabled=false
io.personium.core.cache.box.enabled=false
io.personium.core.cache.schema.enabled=false

# security configurations
io.personium.core.security.secret16=gv7hpmmf5siwj5by
io.personium.core.security.auth.password.salt=voAbizvF
io.personium.core.security.dav.encrypt.enabled=false

# Davlimit configurations
io.personium.core.dav.childresource.maxnum=30
io.personium.core.dav.depth.maxnum=5

# OData $links configurations
io.personium.core.odata.links.NtoN.maxnum=40
```

**/personium_engine/src/main/resources/personium-unit-config.properties**

```
#################################################
# personium-engine configurations
#################################################
# general configurations
io.personium.core.masterToken=personiumio

io.personium.core.es.cluster.name=【elasticsearch.ymlで設定した値】

io.personium.core.blobStore.root=/personium/personium_nfs/personium-core/dav

# security configurations
io.personium.core.security.secret16=secret167pm5m4y6
```

> ※Eclipse上でmavenやpom.xml関連のビルドエラーが出ている場合、  
以下に記載した手順を実施してください。

1. コマンドプロンプト等でpom.xmlが存在するディレクトリに移動します。  

1. 以下のコマンドを実行します。

    ```
    mvn clean package -DskipTests
    ```

    コマンドがSUCCESSで終了しなかった場合は原因を取り除いて再度実施します。  

1. Eclipse上で対象プロジェクトを右クリック→[Maven]→[プロジェクトの更新]→[OK]を選択します。


## 起動確認方法
コマンドは管理者権限を持つユーザで実行してください。
### Elasticsearch起動確認

コマンドプロンプト等でElasticsearchのインストールディレクトリに移動します。

```
> cd Elasticsearchインストールディレクトリ
```

以下コマンドを実行します。

```
> bin\service install
> bin\service start
```

ブラウザでhttp://localhost:9200にアクセスします。  

以下のように出力されると起動成功となります。

```
{
  "name" : "MODAM",
  "cluster_name" : "【elasticsearch.ymlで設定した値】",
  "cluster_uuid" : "VM2fJ2hXQRSI4PsYhJBtLA",
  "version" : {
    "number" : "5.6.14",
    "build_hash" : "f310fe9",
    "build_date" : "2018-12-05T21:20:16.416Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

### ActiveMQ起動確認

コマンドプロンプト等でActivemqのインストールディレクトリに移動します。

```
> cd Activemqインストールディレクトリ
```

以下コマンドを実行します。

```
> bin\activemq start
```

ブラウザでhttp://localhost:8161にアクセスします。  
管理画面が表示されると起動成功となります。  

### 開発用サーバ起動確認
Eclipse上で[personium-core]プロジェクトを右クリック→[実行]→[サーバで実行]を選択します。  

初回は新規サーバの定義でTomcat9のサーバを追加して実行します。  

> ※Tomcat9の起動でタイムアウト時間を超えて起動エラーになることがあります。  
EclipseでTomcat起動時のタイムアウト時間を変更してください。


ブラウザでhttp://localhost:8080/personium-core/にアクセスします。  
以下のように出力されると起動成功となります。

```
{"unit":{"path_based_cellurl_enabled":true,"url":"http:\/\/localhost\/"}}
```

### Nginx起動確認

コマンドプロンプト等でNginxインストールディレクトリに移動します。

```
> cd Nginxインストールディレクトリ
```

以下のコマンドを実行します。

```
> start nginx
```

ブラウザでhttp://localhostにアクセスします。  
以下のように出力されると起動成功となります。

```
{"unit":{"path_based_cellurl_enabled":true,"url":"http:\/\/localhost\/"}}
```
