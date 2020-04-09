---
title: サーバーのスケールアウト
descriptions: Optimizations that can be done to serve more users.
menu:
  docs:
    weight: 100
    parent: admin
---

## 並列実行の管理 {#concurrency}

Mastodonは3タイプのプロセスを持ちます

* Web \(Puma\)
* Streaming API
* バックグラウンド処理 \(Sidekiq\)

### Web \(Puma\) {#web}


Webプロセスは、ほとんどのアプリケーションに対して存続時間の短いHTTPリクエストを処理します。次の環境変数で制御できます：

* `WEB_CONCURRENCY` ワーカープロセスの数を制御
* `MAX_THREADS` プロセスあたりのスレッド数を制御

スレッドは親プロセスのメモリをシェアします。異なるプロセスは自身のためのメモリを確保しますが、コピーオンライトによってメモリを共有します。スレッド数を増やすとCPU使用率が上昇し、プロセス数を増やすとメモリを多く消費するようになります。

それらの値は、同時に処理できるHTTPリクエストの数に影響します。

スループットに関しては、スレッドを増やすよりプロセスを増やした方が効果的です。

### Streaming API {#streaming}

ストリーミングAPIは、クライアントがリアルタイムの更新を受信するための、持続時間の長いHTTP接続とWebSocket接続を処理します。次の環境変数で制御できます：

* `STREAMING_CLUSTER_NUM` ワーカープロセスの数を制御
* `STREAMING_API_BASE_URL` Streaming APIのBase URLを設定

ひとつのプロセスでかなり多くの接続を処理できます。Streaming APIは、必要に応じて別のサブドメインでホストすることができ、Nginxプロキシのオーバヘッドを回避することができます。


### バックグラウンド処理 \(Sidekiq\) {#sidekiq}

Mastodonの多くのタスクはバックグラウンド処理に委任されており、HTTPリクエストの高速性を保証し、HTTPリクエストの中断がタスクの実行に影響を与えないようにしています。Sidekiqはスレッド数を設定可能なひとつのプロセスです。


#### スレッド数 {#sidekiq-threads}

Webプロセスのスレッド数は、エンドユーザーに対するMastodonインスタンスの応答性に影響します。バックグラウンド処理に割り当てられたスレッド数は、投稿の配信速度に影響します。たとえばメールの送信など。

スレッド数の設定は環境変数によって設定されるわけでなく、Sidekiq実行時のコマンドライン引数で設定できます。たとえば：

```bash
bundle exec sidekiq -c 15
```

これでSidekiqは15スレッドで実行されます。各スレッドはデータベース接続を必要とすることに注意してください。つまり、データベースプールは、すべてのスレッドを受け入れられるだけの大きさである必要があります。データベースプールのサイズは`DB_POOL`環境変数で設定できます。これは少なくともスレッドの数と同じでなければなりません。


#### キュー {#sidekiq-queues}

Sidekiqは異なる種類のキューを重要度によって使い分けています。重要度は、それぞれのキューが動作しなかった場合、ローカルユーザーのユーザーエクスペリエンスに与える影響度の大きさによって定義されています：


| Queue | Significance |
| :--- | :--- |
| `default` | ローカルユーザーに影響するすべてのタスク |
| `push` | 他のサーバーへの配送 |
| `mailers` | メールの配信 |
| `pull` | 他のサーバーからの情報取得 |

デフォルトのキューとそれらの優先度については`config/sidekiq.yml`に保存されていますが、Sidekiqのコマンドライン引数によって上書きできます。たとえば：

```bash
bundle exec sidekiq -q default
```

とすれば、`default`キューが動きます。

Sidekiqのキューの処理は、最初のキューからタスクの有無を確認し、タスクがなければ次のキューを確認しに行きます。つまり、最初のキューがいっぱいになってしまうと、以降のキューの処理が遅延します。

解決策として、キューごとにSidekiqプロセスを起動して、並列実行するようにします。そのために、キューごとに引数の異なるsystemdサービスを作成します。


## pgBouncerによるトランザクションプーリング {#pgbouncer}

### PgBouncerが必要になる理由 {#pgbouncer-why}

もしPostgreSQLの利用可能な接続数（デフォルトでは100です）を使い切るようであれば、PgBouncerの導入により解決できます。このドキュメントでは、いくつかの一般的な問題と、Mastodonの適切なデフォルト設定について説明します。

管理画面の"PgHero"で、現在使用されているPostgreSQL接続の数を確認することができます。通常、Mastodonは、Puma、Sidekiq、Streaming APIのスレッド数を足した数の接続を使用します。


### PgBouncerのインストール {#pgbouncer-install}

DebianとUbuntuでは:

```bash
sudo apt install pgbouncer
```

### PgBouncerの設定 {#pgbouncer-config}

#### パスワードの設定 {#pgbouncer-password}

まず、PostgreSQLの`mastodon`ユーザーがパスワードなしで設定されている場合、パスワードを設定する必要があります。

パスワードをリセットする方法は下記の通りです:

```bash
psql -p 5432 -U mastodon mastodon_production -w
```

そして （"password"ではない安全なパスワードを使用して下さい）:

```sql
ALTER USER mastodon WITH PASSWORD 'password';
```

それから `\q` で抜けます。

#### userlist.txtの設定 {#pgbouncer-userlist}

`/etc/pgbouncer/userlist.txt` を編集します。

後でuserlist.txtにおいてユーザー名・パスワードを指定する場合は、userlist.txt の値は実際のPostgreSQLのロールに対応する必要はありません。
ユーザー名とパスワードは任意に設定できますが、利便性のために「実際の」資格情報を再利用できます。`mastodon`ユーザーを`userlist.txt`に追加します：


```text
"mastodon" "md5d75bb2be2d7086c6148944261a00f605"
```

ここでは、md5スキームを使用しています。md5パスワードは`password + username` の md5sumに`md5` という文字列を付加したものです。たとえば、ユーザー`mastodon`とパスワード`password`のMD5ハッシュを取得するには、次のようにします：

```bash
# ubuntu, debian, etc.
echo -n "passwordmastodon" | md5sum
# macOS, openBSD, etc.
md5 -s "passwordmastodon"
```

そして、頭に`md5`を付けます。

また、PgBouncer管理でデータベースにログインするための`pgbouncer`管理ユーザーを作成することもできます。以下は`userlist.txt`のサンプルです：

```text
"mastodon" "md5d75bb2be2d7086c6148944261a00f605"
"pgbouncer" "md5a45753afaca0db833a6f7c7b2864b9d9"
```

どちらもパスワードは `password`です。

#### pgbouncer.ini の設定 {#pgbouncer-ini}

`/etc/pgbouncer/pgbouncer.ini` を編集します。

`[databases]`の下に行を追加し、接続したいPostgresデータベースを列挙します。以下は同じユーザー名・パスワード・データベース名を使用してPostgresデータベースに接続する例です：

```text
[databases]
mastodon_production = host=127.0.0.1 port=5432 dbname=mastodon_production user=mastodon password=password
```
`listen_addr` と `listen_port`はPgBouncerに接続するためのアドレスとポート番号です。デフォルトのままでよいでしょう：

```text
listen_addr = 127.0.0.1
listen_port = 6432
```

`auth_type`として`md5`を設定します。（`userlist.txt`でmd5フォーマットを使用していると仮定しています）：

`pgbouncer`が管理者であることを確認します：

**次の部分はとても重要です！** デフォルトのプーリングモードはセッションベースですが、Mastodonではトランザクションベースに設定します。つまり、Postgresの接続はトランザクションが開始されたときに作成され、トランザクションが完了すると削除されます。そのため、`pool_mode` を `session` から`transaction`に変更してください：

続いて、`max_client_conn`ではPgBouncer自身が受け付ける接続数を設定し、`default_pool_size`はPostgresの接続数の制限です。（PgHeroではPgBouncerを知らないので、接続数として`default_pool_size`が表示されます。）

開始時はデフォルト値十分です。いつでも大きくできます：

```text
max_client_conn = 100
default_pool_size = 20
```

設定を適用するために、PgBouncerのリロードもしくは再起動を忘れないでください：

```bash
sudo systemctl reload pgbouncer
```

#### すべてがうまく動作しているか確認する {#pgbouncer-debug}
#### Debugging that it all works {#pgbouncer-debug}

PgBouncerへの接続は、Postgresに直接接続する時のように行えます：

```bash
psql -p 6432 -U mastodon mastodon_production
```

そしてパスワードを入力してログインします。

PgBouncerのログをチェックする場合は以下のようにします：

```bash
tail -f /var/log/postgresql/pgbouncer.log
```

#### MastodonがPgBouncerを使うようにする {#pgbouncer-mastodon}

最初に`.env.production`ファイルを開き次のように設定します： 

```bash
PREPARED_STATEMENTS=false
```

トランザクションベースのプーリングを使用するため、プリペアドステートメントを使用することができません。

続いて、Mastodonが5432番ポート（Postgres）の代わりに 6432番ポート（PgBouncer）を使うよう構成します：

```bash
DB_HOST=localhost
DB_USER=mastodon
DB_NAME=mastodon_production
DB_PASS=password
DB_PORT=6432
```

{{< hint style="warning" >}}
pgBouncerを介して`db:migrate` を実行することができません。しかし簡単に回避することができます。PostgresとpgBouncerが同一ホストで動いているのなら、タスクを実行するときに`RAILS_ENV=production`と同じように`DB_PORT=5432`と指定するだけです。たとえば： `RAILS_ENV=production DB_PORT=5432 bundle exec rails db:migrate` \(データベースサーバーが違う場合も、`DB_HOST`を指定できます。\)
{{< /hint >}}

#### PgBouncerの管理 {#pgbouncer-admin}


再起動する簡単な方法は：

```bash
sudo systemctl restart pgbouncer
```

しかしpgBouncerの管理者ユーザーをセットアップした場合は、管理者として接続できます：

```bash
psql -p 6432 -U pgbouncer pgbouncer
```

続いて:

```sql
RELOAD;
```

終わったら `\q` で抜けられます。


## Separate Redis for cache {#redis}

Redisはアプリケーション全体で広く使われていますが、いくつかの重要な用途にも使われています。ホームタイムライン、リストフィード、Sidekiqキュー、ストリーミングAPIなどはRedisによってバックアップされており、これは重要なデータです。（PostgreSQLデータベースと違い、失ってしまっても生き残ることはできます。）しかし、Redisは揮発性キャッシュとしても使われており、スケールアップする段階で、Redisのキャパシティに不安がある場合は、キャッシュ用に別のRedisデータベースを使うことができます。環境変数では、`CACHE_REDIS_URL`や`CACHE_REDIS_HOST`、`CACHE_REDIS_PORT`などを個別に設定が可能です。指定しない部分は、キャッシュプレフィックスを使用しない場合と同じ値にフォールバックします。

Redisデータベースを設定すると、基本的にはディスクへの書き込み頻度を抑えることができます。というのも、再起動時にデータが失われても問題にならず、その分ディスクI/Oを節約できるからです。また、最大メモリ制限やキーの退避ポリシー設定を追加することもできます。詳しくは次のガイドを参照してください： [Using Redis as an LRU cache](https://redis.io/topics/lru-cache)


## Read-replicas {#read-replicas}

PostgreSQLサーバーの負荷を減らしたいとき、ホットストリーミングレプリケーション（リードレプリカ）を設定したいと思うかもしれません。[このガイドを参照してください](https://cloud.google.com/community/tutorials/setting-up-postgres-hot-standby)。Mastodonでは、次のような方法でレプリカを活用することができます：

* Streaming APIサーバーは書き込みを一切行わないので、そのままレプリカに接続しても問題ありません。しかし、最初からデータベースへの問い合わせの頻度は低いので、効果は薄いです。
* WebプロセスとSidekiqプロセスでMakaraドライバを使用して、書き込みはマスターデータベースに、読み込みはレプリカに行くようにします。その話をしてみましょう。

`config/database.yml`を編集し、`production`セクションを以下のように置換する必要があります：

```yaml
production:
  <<: *default
  adapter: postgresql_makara
  prepared_statements: false
  makara:
    id: postgres
    sticky: true
    connections:
      - role: master
        blacklist_duration: 0
        url: postgresql://db_user:db_password@db_host:db_port/db_name
      - role: slave
        url: postgresql://db_user:db_password@db_host:db_port/db_name
```

URLがPostgreSQLサーバーの場所を挿していることを確認してください。複数のレプリカを追加することもできます。例えば、"mastodon"はmasterに、"mastodon\_replica"はレプリカに接続するように設定して、インストールされているpgBouncerをデータベース名に基づいて2つの異なるサーバーに接続すことができます。このような設定にはたくさんの可能性があります！ Makaraの詳細については、[ドキュメントを参照してください](https://github.com/taskrabbit/makara#databaseyml)。


{{< hint style="warning" >}}
Sidekiqはリードレプリカを確実に使用することができません。ほんのわずかなレプリケーションの遅れでも、キューに入っているレコードを見つけられず、ジョブの失敗につながるからです。
{{< /hint >}}

