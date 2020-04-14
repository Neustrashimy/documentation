---
title: 開発環境のセットアップ
description: Instructions on how to start developing for Mastodon.
menu:
  docs:
    weight: 20
    parent: dev
---

{{< hint style="danger" >}}
This page is under construction.
{{< /hint >}}

### セットアップ {#setup}

`bundle install` と `yarn install` を、プロジェクトディレクトリで実行してください。

開発環境では、Mastodonは現在サインインしているユーザーとして使う、通常はそのまま使える`ident`認証を使用します。`mastodon_development` と `mastodon_test` データベースを作成してスキーマを生成するために、`rails db:setup`コマンドを実行する必要があります。それから、 `db/seed.rb`で定義されたシードデータを `mastodon_development` に作成します。シードデータは、認証情報`admin@localhost:3000` / `mastodonadmin`を持った管理者アカウントのみです。

> Mastodonはデフォルトで3000番ポートで実行されることに注意してください。もしポート番号を変更していたら、生成される管理者アカウントはその番号を使用します。


### 実行 {#running}

Mastodonをフルセットで動作させるためには、複数のプロセスを動作させる必要がありますが、選択的に止めることができます。すべての機能をひとつのコマンドで実行するためには、`gem install foreman --no-document` で Foremanをインストールし、Mastodonディレクトリ内で次のように使用します：

```text
foreman start
```

これで`Procfile.dev`に定義されたプロセスを実行します。次のものが含まれます： Railsサーバー、Webpackサーバー、ストリーミングAPIサーバー、そしてSidekiqです。もちろん、必要に応じて、それらの機能をスタンドアロンで実行することができます。


### テスト {#testing}

| Command | Description |
| :--- | :--- |
| `rspec` | Rubyテストスイートを実行する |
| `yarn run test` | JavaScriptテストスイートを実行する |
| `rubocop` | Rubyコードがコーディング規約に準拠しているかチェックする |

