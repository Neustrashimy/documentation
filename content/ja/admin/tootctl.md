---
title: 管理者 CLI を使用する
description: tootctl commands that can be run from the CLI.
menu:
  docs:
    weight: 60
    parent: admin
---
---

Mastodonのコマンドラインインタフェイスは、Mastodonディレクトリの `bin` にある `tootctl` というプログラムです。どの環境で実行するか、環境変数 `RAILS_ENV` で指定する必要があります。ローカルマシンで開発をしているのではなければ、 `RAILS_ENV=production` を指定する必要があります。もし他の環境（development、testing、stagingなど）を使うつもりがなければ、 `.bashrc` に追加すると便利です。たとえば：

```bash
echo "export RAILS_ENV=production" >> ~/.bashrc
```

そうすれば、毎回インラインで指定する必要がなくなります。でなければ、`tootctl`の通常の呼び出し方は次の通りです。Mastodonのプログラムがチェックアウトされた場所 `/home/mastodon/live` で実行してください：

```bash
cd /home/mastodon/live
RAILS_ENV=production bin/tootctl help
```

## Base CLI

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/cli.rb" caption="lib/cli.rb" >}}

### `tootctl self-destruct` {#self-destruct}

すべての既知のサーバーに`account delete`アクティビティをブロードキャストすることにより、サーバーをフェデレーションから削除する。これにより、他のサーバーにキャッシュを残さず "clean exit" することができる。このコマンドは対話式で、2回の確認がある。

実際にはローカルのデータは削除されません。なぜならばデータベースを直接消したり、VPSそのものを削除した方が高速だからです。このコマンドを実行した後にインスタンスの使用を強行してしまうと、状態の不一致が発生し、予期せぬ不具合やフェデレーションの問題が発生する可能性があります。


{{< hint style="danger" >}}
**このコマンドを実行する前に、あなたがなにをしようとしているのか正しく理解してください。** この作業は復元不能で、かつ長時間かかります。コマンドが終了すると、サーバーは"壊れた状態"になります。実行のためにSidekiqプロセスを使用します。キューがすべて完了するまでサーバーをシャットダウンしないでください。
{{< /hint >}}

**Version history:**
2.8.0 - 追加

| Option | Description |
| :--- | :--- |
| `--dry_run` | 期待される結果のみ出力し、実際には何も行わない。 |

### `tootctl --version` {#version}

現在動作中のMastodonインスタンスのバージョンを表示する。

**Version history:**
2.7.0 - 追加

## Accounts CLI {#accounts}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/accounts_cli.rb" caption="lib/mastodon/accounts\_cli.rb" >}}

### `tootctl accounts rotate` {#accounts-rotate}

新しいRSAキーを生成し、ブロードキャストします。セキュリティメンテナンスの一環として行われます。

**Version history:**
2.5.0 - 追加

| Option | Description |
| :--- | :--- |
| `USERNAME` | アカウントのローカルユーザー名 |
| `--all` | USERNAMEの代わりに指定すると、全てのローカルアカウントに対して実行する |

### `tootctl accounts create` {#accounts-create}

与えられた USERNAMEと --email で、新しいユーザーを作成します。

**Version history:**
2.6.0 - 追加

| Option | Description |
| :--- | :--- |
| `USERNAME`      | 新しいアカウントのローカルユーザー名。必須。 |
| `--email EMAIL` | ユーザーのメールアドレス。必須。 |
| `--confirmed`   | 確認メールを送らず、アカウントを即時有効にする。 |
| `--role ROLE`   | 新しいアカウントの役割。`user`、`moderator`、`admin`のいずれかを指定。規定値は`user`。 |
| `--reattach`    | 削除済みの古いUSERNAMEを再利用する。 |
| `--force`       | すでに存在するアカウントを強制的に削除し、新しいアカウントとして設定する。|

### `tootctl accounts modify` {#accounts-modify}

既存ユーザーアカウントの役割、ステータス、承認状態、二段階認証の有無を設定する。

**Version history:**
2.6.0 - 追加

| Option | Description |
| :--- | :--- |
| `USERNAME` | ローカルユーザー名。必須。 |
| `--role ROLE` | アカウントの役割を`user`、`moderator`、`admin`のいずれかに設定。 |
| `--email EMAIL` | ユーザーのメールアドレスを EMAIL に設定。 |
| `--confirm` | 確認メールを送らない。--email オプションと同時に指定する。 |
| `--disable` | USERNAME のアカウントをロックする。 |
| `--enable` | USERNAME のアカウントがロックされていれば、ロック解除する。 |
| `--approve` | 承認制の場合、アカウントを承認する。 |
| `--disable_2fa` | 二段階認証を解除し、パスワードによるログインを有効にする。 |

### `tootctl accounts delete` {#accounts-delete}

与えられた USERNAME のアカウントを削除する。

**Version history:**
2.6.0 - 追加

| Option | Description |
| :--- | :--- |
| `USERNAME` | ローカルユーザー名。必須。 |

### `tootctl accounts backup` {#accounts-backup}

与えられた USERNAME のバックアップをリクエストする。バックアップはSidekiqにより非同期で行われ、完了したらユーザーにリンクが記載されたメールが送られる。


**Version history:**
2.6.0 - 追加

| Option | Description |
| :--- | :--- |
| `USERNAME` | ローカルユーザー名。必須。 |

### `tootctl accounts cull` {#accounts-cull}

存在しないリモートアカウントを削除する。データベースにある個々のリモートアカウントに対し、元のサーバーにまだ存在するか問い合わせ、存在しなければデータベースから削除する。直近一週間以内にアクティビティがあったユーザーは、サーバーがダウンした場合に備えて除外される。


**Version history:**
2.6.0 - 追加
2.8.0 - `--dry_run` オプションの追加

| Option | Description |
| :--- | :--- |
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=5. |
| `--dry_run` | 期待される結果のみ出力し、実際には何も行わない。 |

### `tootctl accounts refresh` {#accounts-refresh}

ひとつまたは複数のリモートユーザーのデータを再取得する。

**Version history:**
2.6.0 - 追加

| Option | Description |
| :--- | :--- |
| `USERNAME` | ローカルユーザー名。 |
| `--all` | USERNAMEの代わりに指定すると、全てのリモートアカウントに対して実行する。 |
| `--domain DOMAIN` | USERNAMEの代わりに指定できる。指定された DOMAIN のリモートユーザーに対してのみ実行される。 |
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=5. |
| `--verbose` | タスク実行中に、追加の情報を表示する。 |
| `--dry_run` | 期待される結果のみ出力し、実際には何も行わない。 |

### `tootctl accounts follow` {#accounts-follow}

すべてのローカルアカウントに対し、指定されたローカルアカウントを強制的にフォローさせる。

**Version history:**
2.7.0 - 追加
3.0.0 - ACCTの代わりにUSERNAMEを使うようになった

| Option | Description |
| :--- | :--- |
| `USERNAME` | ローカルユーザー名。 |
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=5。 |
| `--verbose` | タスク実行中に、追加の情報を表示する。 |

### `tootctl accounts unfollow` {#accounts-unfollow}

すべてのローカルアカウントに対し、指定されたアドレスを強制的にアンフォローさせる。

**Version history:**
2.7.0 - 追加

| Option | Description |
| :--- | :--- |
| `ACCT` | `username@domain` 形式のアドレス。 |
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=5。 |
| `--verbose` | タスク実行中に、追加の情報を表示する。 |

### `tootctl accounts reset-relationships` {#accounts-reset-relationships}

指定されたローカルアカウントのフォロー・フォロワー関係を全てリセットする。

**Version history:**
2.8.0 - 追加

| Option | Description |
| :--- | :--- |
| `USERNAME` | ローカルユーザー名。 |
| `--follows` | USERNAMEの全てのフォローを強制的に解除し、それからリフォローする。|
| `--followers` | USERNAMEの全てのフォロワーを解除する。 |

### `tootctl accounts approve` {#accounts-approve}

インスタンス承認制の場合、新規登録者を承認する。

**Version history:**
2.8.0 - 追加

| Option | Description |
| :--- | :--- |
| `USERNAME` | 指定されたユーザー名で保留中のアカウントを承認する。 |
| `--number N` | 直近 N 件の登録を承認する。 |
| `--all` | すべての保留中の登録を承認する。 |

## Cache CLI {#cache}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/cache_cli.rb" caption="lib/mastodon/cache\_cli.rb" >}}

### `tootctl cache clear` {#cache-clear}

キャッシュストレージをクリアする。

**Version history:**
2.8.1 - 追加

### `tootctl cache recount` {#cache-recount}

ハードキャッシュされたカウンターを再集計する。これはデータベースのサイズによって完了までに長い時間を要する。"Accounts"はフォロワー、フォロー、投稿数を更新する。"Statuses"はリプライ、ブースト、お気に入りの数を更新する。

**Version history:**
3.0.0 - 追加

| Option | Description |
| :--- | :--- |
| TYPE | `accounts` もしくは `statuses` のいずれかを指定。 |
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=5。 |
| `--verbose` | タスク実行中に、追加の情報を表示する。 |

## Domains CLI {#domains}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/domains_cli.rb" caption="lib/mastodon/domains\_cli.rb" >}}

### `tootctl domains purge` {#domains-purge}

与えられたDOMAINに属する全てのアカウントを削除する。サスペンドと違い、DOMAINが依然として存在する場合、それに属しているアカウントが再度現れることを意味します。

**Version history:**
2.6.0 - 追加
2.8.0 - `--whitelist_mode` オプション追加
2.9.0 - 絵文字も削除するようになった
3.0.0 - 引数として複数のドメインを受け入れるようになった

| Option | Description |
| :--- | :--- |
| `DOMAIN[...]` |削除するドメイン。複数指定する場合は、スペースで区切る。 |
| `--whitelist_mode` | DOMAINの代わりに指定できる。ドメインをひとつひとつ消すのではなく、ホワイトリストに登録されていないアカウントのユーザーをデータベースから削除する。ホワイトリストモードを有効にし、ホワイトリストを定義した後に使用する。 |
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=5。 |
| `--verbose` | タスク実行中に、追加の情報を表示する。 |
| `--dry_run` | 期待される結果のみ出力し、実際には何も行わない。 |

### `tootctl domains crawl` {#domains-crawl}

MastodonのREST APIを用いて、既知のフェディバースをクロールする。ピアがAPIエンドポイントをサポートしている場合は、統計情報の取得を試みる。STARTが指定されていない場合、コマンドはサーバーのデータベースにある既知のピアでクロールをシードする。全てのサーバー、登録ユーザーの合計、直近7日間のアクティブユーザーの合計、直近7日間の新規ユーザーの合計を連結して返す。

**Version history:**
2.7.0 - 追加
3.0.0 - `--exclude_suspended` オプションを追加

| Option | Description |
| :--- | :--- |
| START | オプション。探索を開始するサーバーを指定する。|
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=50。 |
| `--format FORMAT` | 探索結果の表示方法をコントロールする。`summary` は要約を表示、 `domains` は探索できたピアを改行で区切って表示、 `json`は集計された生データをJSON形式でダンプする。規定値は `summary`。 |
| `--exclude_suspended` | ドメインブロックしたインスタンスを結果に含まない。除外対象にはサブドメインも含む。 |

## Emoji CLI {#emoji}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/emoji_cli.rb" caption="lib/mastodon/emoji\_cli.rb" >}}

### `tootctl emoji import` {#emoji-import}

カスタム絵文字を指定されたパスにある .tar.gz アーカイブファイルからインポートする。 アーカイブには、50KB以下のPNGもしくはGIFファイルを含める。ショートコードとしてファイル名から拡張子を除いたものが使われる。また、ショートコードにはオプションのプレフィックス・サフィックスが追加される。

**Version history:**
2.5.0 - 追加

| Option | Description |
| :--- | :--- |
| `PATH` | 画像が含まれる .tar.gz アーカイブへのパス |
| `--prefix PREFIX` | PREFIX を生成されたショートコードの先頭に付加する。|
| `--suffix SUFFIX` | SUFFIX を生成されたショートコードの末尾に付加する。 |
| `--overwrite` | 既存の絵文字を残すのではなく、同じショートコードを使っている絵文字をすべて置き換える。 |
| `--unlisted` | 絵文字ピッカーには表示せず、ショートコードの直接入力でのみ使用可能とする。 |
| `--category CATEGORY` | 絵文字ピッカーのカテゴリーグループを指定する。|

### `tootctl emoji purge` {#emoji-purge}

すべてのカスタム絵文字を消す。

**Version history:**
3.1.0 - `--remote_only` オプションを追加

| Option | Description |
| :--- | :--- |
| `--remote_only` | これを指定した場合、リモートドメインの絵文字のみ削除する。 |


**Version history:**
2.8.0 - 追加

## Feeds CLI {#feeds}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/feeds_cli.rb" caption="lib/mastodon/feeds\_cli.rb" >}}

### `tootctl feeds build` {#feeds-build}

すべてのユーザーのホームタイムライン・リストフィードを生成する。フィードはデータベースから生成され、Redisのインメモリにキャッシュされる。Mastodonはアクティブユーザーのホームフィードを自動的に管理する。

**Version history:**
2.6.0 - 追加

| Option | Description |
| :--- | :--- |
| `USERNAME` | フィードを再生成するローカルユーザー名。 |
| `--all` | USERNAMEの代わりに指定できる。すべてのローカルアカウントのフィードを再生成する。 |
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=5。 |
| `--verbose` | タスク実行中に、追加の情報を表示する。 |
| `--dry_run` | 期待される結果のみ出力し、実際には何も行わない。 |

### `tootctl feeds clear` {#feeds-clear}

Redisからすべてのホームタイムラインおよびリストフィードをクリアする。

**Version history:**
2.6.0 - 追加

## Media CLI {#media}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/media_cli.rb" caption="lib/mastodon/media\_cli.rb" >}}

### `tootctl media remove` {#media-remove}

他のサーバーからコピーされたメディアファイルのローカルキャッシュを削除する。

**Version history:**
2.5.0 - 追加
2.6.2 - 確保された空き容量を表示するようになった

| Option | Description |
| :--- | :--- |
| `--days` | どの程度古いメディアファイルを削除対象とするか。デフォルトは 7 (日)。 |
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=5。 |
| `--verbose` | タスク実行中に、追加の情報を表示する。 |
| `--dry_run` | 期待される結果のみ出力し、実際には何も行わない。 |

### `tootctl media remove-orphans` {#media-remove-orphans}

どこにも属していないメディアファイルをスキャンし、削除する。いくつかのストレージプロバイダはオブジェクトをリスト表示するために料金がかかることに注意すること。また、この操作は個々のファイルごとに処理を行うため、低速になる。

**Version history:**
3.1.0 - 追加

| Option | Description |
| :--- | :--- |
| `--start_after` | 指定されたPaperclipの添付キーからループを開始する。このオプションは処理を中断した場合に使用する。 |
| `--dry_run` | 期待される結果のみ出力し、実際には何も行わない。 |

### `tootctl media refresh` {#media-refresh}

他のサーバーからメディアファイルを再取得する。メディアファイルのソースを --status、--account、--domain のいずれかから指定する必要がある。もしメディアファイルがすでにデータベース上に存在する場合、 --force を使わない限り上書きされません。

**Version history:**
3.0.0 - 追加
3.0.1 - `--force` オプションが追加され、すでに存在するメディアファイルはデフォルトでスキップされるようになった

| Option | Description |
| :--- | :--- |
| `--account ACCT` | `username@domain` 形式のアカウントのハンドル文字列 |
| `--domain DOMAIN` | FQDN 文字列 |
| `--status ID` | ローカルのStatus ID |
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=5。 |
| `--verbose` | タスク実行中に、追加の情報を表示する。 |
| `--dry_run` | 期待される結果のみ出力し、実際には何も行わない。 |
| `--force` | 強制的にリモートのリソースを再ダウンロードし、ローカルのデータを上書きする。 |

### `tootctl media usage` {#media-usage}

Mastodonによって試用されているディスク容量を計算する。

**Version history:**
3.0.1 - 追加

### `tootctl media lookup` {#media-lookup}

media URLを入力するプロンプトを表示し、メディアがどこに表示されているか検索する。

**Version history:**
3.1.0 - 追加

## Preview Cards CLI {#preview_cards}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/preview_cards_cli.rb" caption="lib/mastodon/preview\_cards\_cli.rb" >}}

### `tootctl preview_cards remove` {#preview_cards-remove}

ローカルのプレビューカードのサムネイルを削除する。

**Version history:**
3.0.0 - 追加

| Option | Description |
| :--- | :--- |
| `--days` | どの程度古いメディアファイルを削除対象とするか。デフォルトは180（日）。（14日以内のプレビューカードを削除することは推奨されません。なぜなら2週間以内に再投稿されたプレビューカードは再取得されないためです。） |
| `--concurrency N` | このタスクにいくつのワーカーを割り当てるか。規定値は N=5。 |
| `--verbose` | タスク実行中に、追加の情報を表示する。 |
| `--dry_run` | 期待される結果のみ出力し、実際には何も行わない。 |
| `--link` | リンクタイプのプレビューカードのみ対象とし、動画・画像タイプのプレビューカードを残す。 |

## Search CLI {#search}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/search_cli.rb" caption="lib/mastodon/search\_cli.rb" >}}

### `tootctl search deploy` {#search-deploy}

ElasticSearchのインデックスを作成・更新し、投入する。もしElasticSearchが空であれば、このコマンドは必要なインデックスを作成し、データベースからデータをインポートする。このコマンドは、元となるスキーマが更新された場合にもインデックスをアップグレードする。

**Version history:**
2.8.0 - 追加
3.0.0 - 並列実行のため `--processes` オプションが追加された

| Option | Description |
| :--- | :--- |
| `--processes N` | コマンドの並列実行数。デフォルトは N=2。`auto`を指定すると、利用可能なCPUに基づいた値を使用する。 |

## Settings CLI {#settings}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/settings_cli.rb" caption="lib/mastodon/settings\_cli.rb" >}}

### `tootctl settings registrations open` {#settings-registrations-open}

新規登録を可能にする。

**Version history:**
2.6.0 - 追加

### `tootctl settings registrations close` {#settings-registrations-close}

新規登録を不可能にする。

**Version history:**
2.6.0 - added

## Statuses CLI {#statuses}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/statuses_cli.rb" caption="lib/mastodon/statuses\_cli.rb" >}}

### `tootctl statuses remove` {#statuses-remove}

ローカルの誰からもフォローされていない外部ユーザーの投稿や、リプライやその他のアクションが行われていないものなど、どこからも参照されていない投稿をデータベースから削除する。
これは計算量の多い処理であり、データベースに一時的なインデックスを作成し、終わったらインデックスを削除する。

**Version history:**
2.8.0 - 追加

| Option | Description |
| :--- | :--- |
| `--days` | どの程度古い投稿を削除対象とするか。デフォルトは 90 (日)。 |

