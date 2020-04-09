---
title: OAuth スコープ
description: Defining what you have permission to do with the API
menu:
  docs:
    weight: 10
    parent: api
---

## OAuth スコープ

APIはアクセススコープに分かれています。スコープは段階的で、`read`のアクセス権を持っている場合、自動的に`read:accounts`へのアクセス権も得ていることになります。**アプリケーションには最低限のリクエストをすることをお勧めします。**

同時に複数のスコープをリクエストすることができます： `scopes`パラメータを付けた状態でのアプリ作成時、もしくは`scope`クエリパラメータを付けた状態での認証時（スコープはスペースで区切る）。


{{< hint style="info" >}}
`scope`と`scopes`の違いに注意してください。`scope`は標準的なOAuthのパラメータ名で、OAuthメソッド内で使用されています。Mastodon自身のREST APIでは、より適切な`scopes` を使っています。
{{< /hint >}}

もし、認証リクエスト時に`scope`を指定しなかったり、アプリ作成時に`scopes`を指定しなかった場合、返されるアクセストークンやアプリは、既定の`read`アクセスを持ちます。

アプリ作成時に保存されるスコープのセットには、認証リクエストでリクエストする全てのスコープが含まれる必要があります。でなければ認証は失敗します。


### Version history {#versions}

- 0.9.0 - read, write, follow
- 2.4.0 - push
- 2.4.3 - granular scopes [https://github.com/tootsuite/mastodon/pull/7929](https://github.com/tootsuite/mastodon/pull/7929)
- 2.6.0 - read:reports が非推奨に \(使用されていないスタブ\) [https://github.com/tootsuite/mastodon/pull/8736/commits/adcf23f1d00c8ff6877ca2ee2af258f326ae4e1f](https://github.com/tootsuite/mastodon/pull/8736/commits/adcf23f1d00c8ff6877ca2ee2af258f326ae4e1f)
- 2.6.0 - write:conversations 追加 [https://github.com/tootsuite/mastodon/pull/9009](https://github.com/tootsuite/mastodon/pull/9009)
- 2.9.1 - Admin スコープ追加 [https://github.com/tootsuite/mastodon/pull/9387](https://github.com/tootsuite/mastodon/pull/9387)
- 3.1.0 - Bookmark スコープ追加

## List of scopes

### `read` {#read}

データの読み込みを許可します。`read`をリクエストすると、下表の左側の列に示される子スコープも許可されます。

### `write` {#write}

データの書き込みを許可します。`write`をリクエストすると、下表の右側の列に示される子スコープも許可されます。

### `follow` {#follow}

リレーションシップへのアクセスを許可します。`follow`をリクエストすると、次の子スコープも許可されます。表では太字で示されます：

* `read:blocks`, `write:blocks`
* `read:follows`, `write:follows`
* `read:mutes`, `write:mutes`

### `push` {#push}

[Web Push API の購読]({{< relref "../methods/notifications/push.md" >}}) を許可します。Mastodon 2.4.0 で追加されました。

### Admin scopes {#admin}

モデレーションAPIに使われます。Mastodon 2.9.1で追加されました。次の詳細なスコープを使用できます（`admin`という単一のスコープがないことに注意して下さい）：

* `admin:read`
  * `admin:read:accounts`
  * `admin:read:reports`
* `admin:write`
  * `admin:write:accounts`
  * `admin:write:reports`

## Granular scopes {#granular}

| read | write |
| :--- | :--- |
| read:accounts | write:accounts |
| **read:blocks** | **write:blocks** |
| read:bookmarks | write:bookmarks |
|  | write:conversations |
| read:favourites | write:favourites |
| read:filters | write:filters |
| **read:follows** | **write:follows** |
| read:lists | write:lists |
|  | write:media |
| **read:mutes** | **write:mutes** |
| read:notifications | write:notifications |
|  | write:reports |
| read:search |  |
| read:statuses | write:statuses |

| admin:read | admin:write |
| :--- | :--- |
| admin:read:accounts | admin:write:accounts |
| admin:read:reports | admin:write:reports |

