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
| `--whitelist_mode` | DOMAINの代わりに指定できる。Can be provided instead of DOMAIN. Instead of purging from a single domain, all accounts from domains that are not whitelisted will be removed from the database. Use this after enabling whitelist mode and defining your whitelist. |
| `--concurrency N` | The number of workers to use for this task. Defaults to 5. |
| `--verbose` | Print additional information while task is processing. |
| `--dry_run` | Print expected results only, without performing any actions. |

### `tootctl domains crawl` {#domains-crawl}

Crawl the known fediverse by using Mastodon REST API endpoints that expose all known peers, and collect statistics from those peers, as long as those peers support those API endpoints. When no START is given, the command uses the server's own database of known peers to seed the crawl. Returns total servers, total registered users, total active users in the last week, and total users joined in the last week.

**Version history:**
2.7.0 - added
3.0.0 - add `--exclude_suspended`

| Option | Description |
| :--- | :--- |
| START | Optionally start from a different domain name. |
| `--concurrency N` | The number of workers to use for this task. Defaults to 50. |
| `--format FORMAT` | Control how results are returned. `summary` will print a summary. `domains` will return a newline-delimited list of all discovered peers. `json` will dump aggregated raw data. Defaults to `summary`. |
| `--exclude_suspended` | Do not include instances that you have suspended in the output. Also includes any subdomains. |

## Emoji CLI {#emoji}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/emoji_cli.rb" caption="lib/mastodon/emoji\_cli.rb" >}}

### `tootctl emoji import` {#emoji-import}

Imports custom emoji from a .tar.gz archive at a given path. The archive should contain PNG or GIF files no larger than 50KB, and the shortcode will be set equal to the filename minus the extension, with optional prefixes and/or suffixes.

**Version history:**
2.5.0 - added

| Option | Description |
| :--- | :--- |
| `PATH` | Path to a .tar.gz archive containing pictures. |
| `--prefix PREFIX` | Add PREFIX to the beginning of generated shortcodes. |
| `--suffix SUFFIX` | Add SUFFIX to the end of generated shortcodes. |
| `--overwrite` | Instead of skipping existing emoji, replace them with any discovered emoji with the same shortcode. |
| `--unlisted` | Processed emoji will not be shown in the emoji picker, but will be usable only by their direct shortcode. |
| `--category CATEGORY` | Group the processed emoji under CATEGORY in the picker. |

### `tootctl emoji purge` {#emoji-purge}

Remove all custom emoji.

**Version history:**
3.1.0 - add `--remote_only`

| Option | Description |
| :--- | :--- |
| `--remote_only` | If provided, remove only from remote domains. |


**Version history:**
2.8.0 - added

## Feeds CLI {#feeds}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/feeds_cli.rb" caption="lib/mastodon/feeds\_cli.rb" >}}

### `tootctl feeds build` {#feeds-build}

Build home and list feeds for one or all users. Feeds will be built from the database and cached in-memory with Redis. Mastodon manages home feeds for active users automatically.

**Version history:**
2.6.0 - added

| Option | Description |
| :--- | :--- |
| `USERNAME` | Local username whose feeds will be regenerated. |
| `--all` | Can be provided instead of USERNAME to refresh all local accounts' feeds. |
| `--concurrency N` | The number of workers to use for this task. Defaults to N=5. |
| `--verbose` | Print additional information while task is processing. |
| `--dry_run` | Print expected results only, without performing any actions. |

### `tootctl feeds clear` {#feeds-clear}

Remove all home and list feeds from Redis.

**Version history:**
2.6.0 - added

## Media CLI {#media}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/media_cli.rb" caption="lib/mastodon/media\_cli.rb" >}}

### `tootctl media remove` {#media-remove}

Remove locally cached copies of media attachments from other servers.

**Version history:**
2.5.0 - added
2.6.2 - show freed disk space

| Option | Description |
| :--- | :--- |
| `--days` | How old media attachments have to be before they are removed. Defaults to 7. |
| `--concurrency N` | The number of workers to use for this task. Defaults to 5. |
| `--verbose` | Print additional information while task is processing. |
| `--dry_run` | Print expected results only, without performing any actions. |

### `tootctl media remove-orphans` {#media-remove-orphans}

Scans for files that do not belong to existing media attachments, and remove them. Please mind that some storage providers charge for the necessary API requests to list objects. Also, this operation requires iterating over every single file individually, so it will be slow.

**Version history:**
3.1.0 - added

| Option | Description |
| :--- | :--- |
| `--start_after` | The Paperclip attachment key where the loop will start. Use this option if the command was interrupted before. |
| `--dry_run` | Print expected results only, without performing any actions. |

### `tootctl media refresh` {#media-refresh}

Refetch remote media attachments from other servers. You must specify the source of media attachments with either --status, --account, or --domain. If an attachment already exists in the database, it will not be overwritten unless you use --force.

**Version history:**
3.0.0 - added
3.0.1 - add `--force` and skip already downloaded attachments by default

| Option | Description |
| :--- | :--- |
| `--account ACCT` | String `username@domain` handle of the account |
| `--domain DOMAIN` | FQDN string |
| `--status ID` | Local numeric ID of the status in the database. |
| `--concurrency N` | The number of workers to use for this task. Defaults to 5. |
| `--verbose` | Print additional information while task is processing.  |
| `--dry_run` | Print expected results only, without performing any actions. |
| `--force` | Force redownload the remote resource and overwrite the local attachment. |

### `tootctl media usage` {#media-usage}

Calculate disk space consumed by Mastodon.

**Version history:**
3.0.1 - added

### `tootctl media lookup` {#media-lookup}

Prompts for a media URL, then looks up where the media is displayed.

**Version history:**
3.1.0 - added

## Preview Cards CLI {#preview_cards}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/preview_cards_cli.rb" caption="lib/mastodon/preview\_cards\_cli.rb" >}}

### `tootctl preview_cards remove` {#preview_cards-remove}

Remove local thumbnails for preview cards.

**Version history:**
3.0.0 - added

| Option | Description |
| :--- | :--- |
| `--days` | How old media attachments have to be before they are removed. Defaults to 180. \(NOTE: it is not recommended to delete preview cards within the last 14 days, because preview cards will not be refetched unless the link is reposted after 2 weeks from last time.\) |
| `--concurrency N` | The number of workers to use for this task. Defaults to 5. |
| `--verbose` | Print additional information while task is processing. |
| `--dry_run` | Print expected results only, without performing any actions. |
| `--link` | Only delete link-type preview cards; leave video and photo cards untouched. |

## Search CLI {#search}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/search_cli.rb" caption="lib/mastodon/search\_cli.rb" >}}

### `tootctl search deploy` {#search-deploy}

Create or update an ElasticSearch index and populate it. If ElasticSearch is empty, this command will create the necessary indices and then import data from the database into those indices. This command will also upgrade indices if the underlying schema has been changed since the last run.

**Version history:**
2.8.0 - added
3.0.0 - add `--processes` for parallelization

| Option | Description |
| :--- | :--- |
| `--processes N` | Parallelize execution of the command. Defaults to N=2. Can also specify `auto` to derive a number based on available CPUs. |

## Settings CLI {#settings}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/settings_cli.rb" caption="lib/mastodon/settings\_cli.rb" >}}

### `tootctl settings registrations open` {#settings-registrations-open}

Opens registrations.

**Version history:**
2.6.0 - added

### `tootctl settings registrations close` {#settings-registrations-close}

Closes registrations.

**Version history:**
2.6.0 - added

## Statuses CLI {#statuses}

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/lib/mastodon/statuses_cli.rb" caption="lib/mastodon/statuses\_cli.rb" >}}

### `tootctl statuses remove` {#statuses-remove}

Remove unreferenced statuses from the database, such as statuses that came from relays or from users who are no longer followed by any local accounts, and have not been replied to or otherwise interacted with.

This is a computationally heavy procedure that creates extra database indices before commencing, and removes them afterward.

**Version history:**
2.8.0 - added

| Option | Description |
| :--- | :--- |
| `--days` | How old statuses have to be before they are removed. Defaults to 90. |

