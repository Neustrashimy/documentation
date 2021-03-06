---
title: 技術概要
description: A description of Mastodon's architecture.
menu:
  docs:
    weight: 10
    parent: dev
---

{{< hint style="warning" >}}
このページは工事中です！
{{< /hint >}}

Mastodonは、フロントエンドにReact.jsを使用した Ruby on Railsアプリケーションです。これらのフレームワークの標準的な手法に従っているため、RailsやReact.jsにすでに精通している場合は驚くべきことではありません。

開発環境でMastodonを使用する最良の方法は、DockerあるいはVagrantを使用するのではなく、システムにすべての依存関係をインストールすることです。Ruby、Node.js、PostgreSQL、Redisが必要になりますが、これらはRailsアプリケーションでは標準的な依存関係のセットです。

### 環境 {#environments}

環境（environment）とは、特定のユースケースを対象とした一連の設定値です。環境は次の通りです：開発（development）、プログラムを変更したい場合。テスト（test）、自動化された一連のテストを実行したい場合。ステージング（staging）、エンドユーザーにコードをプレビューしたい場合。そして本番（production）、エンドユーザーに提供する場合。Mastodonには開発、テスト、本番の構成が付属しています。

`RAILS_ENV` のデフォルト値は `development`です。そのため、開発モードでMastodonを実行する場合は特別な設定は必要ありません。実際、Mastodonのすべての設定には開発環境向けのデフォルト値があるため、何かをカスタマイズする必要がない限り、`.env`ファイルを必要としません。開発環境と本番環境の動作が異なる点は次の通りです：

* Rubyコードは更新されたら自らリロードします。つまり変更を確認するためにRailsサーバープロセスを再起動する必要はありません。
* 遭遇したすべてのエラーのスタックトレースは隠されずにブラウザーに出力されます。
* Webpackは継続的に実行され、フロントエンドファイルが更新されると、JSおよびCSSアセットを再コンパイルし、ページを自動的にリロードします。
* デフォルトでキャッシュが無効になります。
* `db:seed`を実行すると、自動的に管理者アカウントのメールアドレスは`admin@localhost:3000`、パスワードは`mastodonadmin`に設定されます。

Mastodonで配布されているDocker構成は、本番環境向けに最適化されているため、開発には適しません。一方、Vagrant構成は開発用であり、本番用ではありません。