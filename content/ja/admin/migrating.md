---
title: 新しいサーバーへ移行
description: Copying your Mastodon installation to a new server without losing anything.
menu:
  docs:
    weight: 90
    parent: admin
---

時により、様々な理由で、Mastodonインスタンスを他のサーバーに移行したいと思うことがあるでしょう。幸い、その手順はさほど難しくありませんが、ダウンタイムが発生します。


{{< hint style="info" >}}
これはUbuntu Server向けに書かれています。他の環境では手順が異なるかもしれません。
{{< /hint >}}

## 基本的な手順 {#basic-steps}

1. [Production Guide]({{< relref "install.md" >}})を読んで、新しいMastodonサーバーをセットアップします。\(ただし、 `mastodon:setup` は実行しないでください\)
2. 古いサーバーで動いているMastodonを止めます\(たとえば `systemctl stop 'mastodon-*.service'` コマンドを用いて\)。
3. 後述の手順に従い、PostgreSQLデータベースをダンプ＆リストアします。
4. 後述の手順に従い、`system/`のファイルをコピーします。\(ただし、S3などを使っている場合はこの手順を飛ばせます。\)
5. `.env.production`ファイルをコピーします。
6. `RAILS_ENV=production bundle exec rails assets:precompile`を実行し、Mastodonをコンパイルします。
7. `RAILS_ENV=production ./bin/tootctl feeds build`を実行し、各ユーザーのホームタイムラインを再構築します。
8. 新しいサーバーでMastodonを動かします。
9. DNS設定を更新し、新しいサーバーを指すようにします。
10. Nginx設定をコピーもしくは更新し、必要に応じてLet's Encryptの証明書を再取得します。
11. 新しいサーバーをエンジョイしてください！

## 詳細な手順 {#detailed-steps}

### どのデータの移行が必要か {#what-data-needs-to-be-migrated}

おおまかには、以下のようなファイルをコピーする必要があります：

* ユーザーがアップロードした画像や動画が格納されている`~/live/public/system`ディレクトリ。\(ただし、S3などを使っている場合は不要です\)
* PostgreSQLデータベース。 \([pg\_dump](https://www.postgresql.org/docs/9.1/static/backup-dump.html)コマンドを使用します\)
* サーバー設定やシークレットが保存されている `~/live/.env.production` ファイル。

さほど重要ではありませんが、便宜上、以下のファイルもコピーすることをお勧めします：

* Nginx設定ファイル \(`/etc/nginx/sites-available/default`以下にあります\)
* Systemd設定ファイル \(`/etc/systemd/system/mastodon-*.service`\)。サーバーのチューニングやカスタマイズが含まれます。 
* `/etc/pgbouncer`以下に保存されている、PgBouncer設定ファイル。\(もし使っていれば\)


### PostgreSQLのダンプ＆リストア {#dump-and-load-postgres}

`mastodon:setup`を実行する代わりに、`template0` テンプレートを使用して空のデータベースを作成します（ダンプファイルをリストアするときに便利です。詳しくは[pg\_dump ドキュメント](https://www.postgresql.org/docs/9.1/static/backup-dump.html#BACKUP-DUMP-RESTORE)を参照してください）。

古いサーバー上で、`mastodon`ユーザーとして以下のコマンドを実行します：

```bash
pg_dump -Fc mastodon_production -f backup.dump
```

`backup.dump`ファイルを`rsync`や`scp`などで新しいサーバーに転送します。次に、新しいサーバー上で、`mastodon`ユーザーで新しい空のデータベースを作成します：


```bash
createdb -T template0 mastodon_production
```

それから、インポートします：

```bash
pg_restore -U mastodon -n public --no-owner --role=mastodon \
  -d mastodon_production backup.dump
```

\(もし新しいサーバー上のユーザー名が `mastodon`でない場合、上記の`-U`および`--role`の値を変える必要があります。それにより、サーバー間のユーザー名の差異を吸収できます。\)



### ファイルをコピーする {#copy-files}

これは時間が掛かる作業です。不必要なファイルのコピーを避けるため、`rsync`を使うことを推奨します。古いサーバーの`mastodon`ユーザーで、下記を実行します：


```bash
rsync -avz ~/live/public/system/ mastodon@example.com:~/live/public/system/
```

古いサーバー上で何らかのファイルが変更された場合、これを再実行することができます。

サーバーのシークレットが保存されている `.env.production` ファイルもコピーする必要があります。

オプションですが、nginx・systemd・pgbouncerの設定ファイルをコピーするか、再作成します。


### 移行中の案内 {#during-migration}

`~/live/public/500.html`を編集することで、移行中であることを示す、いい感じのエラーメッセージを表示することができます。

DNSのTTLを小さい値（30～60分）に設定することで、新しいIPアドレスの伝播を早めることができます。


### 移行後は {#after-migrating}

[whatsmydns.net](https://whatsmydns.net/)などを利用して、DNSの伝播状況を確認します。または、`/etc/hosts`を編集することで、一足先に新しいサーバーにアクセスすることが可能です。

