---
title: 全文検索
description: Setting up ElasticSearch to search for statuses authored, favourited, or mentioned in.
menu:
  docs:
    weight: 10
    parent: admin-optional
---

ElasticSearchが使用できるとき、Mastodonは全文検索をサポートします。Mastodonの全文検索はログインしたユーザーが使用でき、自身のトゥート、お気に入り、メンションを検索することができます。意図的にデータベース全体の検索をできなくしています。


## ElasticSearchのインストール {#install}

ElasticSearchはJavaランタイムを必要とします。もしJavaのインストールがまだなら、やってしまいましょう。`root`でログインしていることを確認してください：

```bash
apt install openjdk-8-jre-headless
```

ElasticSearchのオフィシャルリポジトリを追加します

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-6.x.list
apt update
```

ElasticSearchのインストールができるようになりました：

```bash
apt install elasticsearch
```

{{< hint style="warning" >}}
**セキュリティ警告：** デフォルトでは、ElasticSearchはlocalhostのみにバインドされており、外部からはアクセスできません。ElasticSearchがどのアドレスにバインドしているかは`/etc/elasticsearch/elasticsearch.yml`内の`network.host`を見れば確認ができます。認証の仕組みがないため、ElasticSearchにアクセスできる人は誰でもデータにアクセスしたり変更したりすることができます。そのため、アクセスの保護はとても重要です。メインのインストール手順で触れられているように、22・80・443ポートのみを解放しているファイアウォールを設置することが推奨されます。マルチホストの設定をしている場合は、内部トラフィックを保護する方法についても知っておく必要があります。
{{< /hint >}}

ElasticSearchを開始します:

```bash
systemctl enable elasticsearch
systemctl start elasticsearch
```

## Mastodonの設定 {#config}

`.env.production`を編集し、次の値を追加します：

```bash
ES_ENABLED=true
ES_HOST=localhost
ES_PORT=9200
```

もし、複数のMastodonサーバーが同じマシンに同居しており、全てのサーバーで同じElasticSearchのインストールを使用する場合、インデックスを区別するために`REDIS_NAMESPACE`が設定されていることを確認してください。ElasticSearchのインデックスのプレフィックスを上書きする必要がある場合は、`ES_PREFIX`を直接指定することができます。

新たな設定値を保存したら、ElasticSearchのインデックスを作成します：

```bash
RAILS_ENV=production bundle exec rake chewy:upgrade
```

新しい設定を有効にするために、Mastodonプロセスを再起動します：

```bash
systemctl restart mastodon-sidekiq
systemctl reload mastodon-web
```

新しい投稿がElasticSearchのインデックスに書き込まれているはずです。最後の手順は、すべての古いデータをインポートすることです。これは時間がかかります：

```bash
RAILS_ENV=production bundle exec rake chewy:sync
```

{{< hint style="warning" >}}
**互換性情報：** Ruby 2.6.0には既知のバグがあり、上記のタスクが正常に動作しません。2.6.1のような他のバージョンのRubyでは問題ありません。
{{< /hint >}}

