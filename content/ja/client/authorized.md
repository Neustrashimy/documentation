---
title: アカウントにログイン
description: How to obtain authorization from a user and perform actions on their behalf.
menu:
  docs:
    weight: 40
    parent: client
---

## スコープについて {#scopes}

アプリを登録し、ユーザーを認証するとき、生成されたトークンが持っている権限を正しく定義する必要があります。これはOAuthスコープを通じて行われます。各APIメソッドには関連付けられたスコープがあり、認証に使用されているトークンが対応するスコープに対する権限を持っている場合のみ呼び出すことができます。

スコープはサブセットを持っています。アプリを作るとき、`read write follow push`を指定すると、全ての利用可能なスコープを要求できますが、より粒度の細かいスコープを使用して、アプリが最低限必要とするものだけを要求することをおすすめします。[OAuth スコープ](../api/oauth-scopes.md)にスコープの一覧があります。各APIメソッドのドキュメントにおいては、呼び出しに必要なOAuthアクセス権限とスコープも記載してあります。


## **認証フローの例** {#flow}

これは以前までの認証フローに似ていますが、ユーザーからの認証も必要となりました。

### Client ID と secret {#client}

始めに、まだクライアントアプリケーションを登録していない場合は、前のページの[アプリを作る](token.md#creating-our-application)を参照して作成するか、[POST /api/v1/apps](../methods/apps/#create-an-application) を参照してメソッドの詳細なドキュメントを確認してください。アプリのために`client_id` と `client_secret`が必要です。

### ユーザーの認証 {#login}

To authorize a user, request [GET /oauth/authorize](../methods/apps/oauth.md#authorize-a-user) in a browser with the following query parameters:

ユーザーを認証するには、ブラウザから下記のクエリパラメータを付けて、[GET /oauth/authorize](../methods/apps/oauth.md#authorize-a-user) をリクエストします。

```bash
https://mastodon.example/oauth/authorize
?client_id=CLIENT_ID
&scope=read+write+follow+push
&redirect_uri=urn:ietf:wg:oauth:2.0:oob
&response_type=code
```

次の点に注意してください：

* `client_id` と `client_secret` は、アプリ登録時に取得したものを使ってください。
* `scope` must be a subset of our registered app's registered scopes. It is a good idea to only request what you need. See [OAuth Scopes](../api/oauth-scopes.md) for more information.
* `scope` は登録したアプリのスコープのサブセットである必要があります。必要最低限のものを要求するとよいでしょう。詳しくは [OAuth Scopes](../api/oauth-scopes.md) を参照してください。
* `redirect_uri` は、登録したアプリのURIです。この例では"アウトオブバンド"を使っています。これは結果のコードを手動でコピー＆ペーストする必要がありますが、URIを使用してアプリケーションを登録した場合、コードはクエリパラメータ`code`として返され、リクエストハンドラによってログに記録されます。詳しくは、APIメソッドのレスポンスの項を参照してください。

### トークンの取得 {#token}

Now that we have an authorization `code`, let's obtain an access token that will authenticate our requests as the authorized user. To do so, use [POST /oauth/token](../methods/apps/oauth.md#obtain-a-token) like before, but pass the authorization code we just obtained:

認証コード（`code`）を取得できましたので、承認済みユーザーとして、リクエストを認証するアクセストークンを取得しましょう。そのためには、認証コードを [POST /oauth/token](../methods/apps/oauth.md#obtain-a-token) に渡します。


```bash
curl -X POST \
	-F 'client_id=your_client_id_here' \
	-F 'client_secret=your_client_secret_here' \
	-F 'redirect_uri=urn:ietf:wg:oauth:2.0:oob' \
	-F 'grant_type=authorization_code' \
	-F 'code=user_authzcode_here' \
	-F 'scope=read write follow push' \
	https://mastodon.example/oauth/token
```

次の点に注意してください：

* `client_id` と `client_secret` は、アプリ登録時に取得したものを使ってください。
* `redirect_uri` は、アプリ登録時に指定したものを使ってください。
*  `authorization_code`の`grant_type`をリクエストしていますが、これはデフォルトで`read`スコープが設定されています。ただし、ユーザーを承認する時に、特定の `scope` をリクエストしていますので、ここでも同じ値を渡します。
* `code` は1回のみ使用できます。新たにトークンを取得するときは、上記の[ユーザーの認証](authorized.md#authorize-the-user)手順を使ってユーザー認証をもう一度行う必要があります。

このメソッドの戻り値は [Token]({{< relref "../entities/token.md" >}}) エンティティです。`access_token`の値が必要となります。アクセストークンを取得したら、ローカルキャッシュに保存してください。これをリクエストで使う場合は、OAuthが必要なAPI（公開アクセスできないもの）を呼び出すとき、HTTPヘッダーに`Authorization: Bearer ...` を追加します。取得した認証情報の検証のため、[GET /api/v1/accounts/verify\_credentials](../methods/accounts/#verify-account-credentials) を呼び出します：

```bash
curl \
	-H 'Authorization: Bearer our_access_token_here' \
	https://mastodon.example/api/v1/accounts/verify_credentials
```

トークンを取得し、正しくリクエストを整形できていれば、 `source`パラメータが含まれた詳細な [Account]({{< relref "../entities/account.md" >}}) エンティティが返されます。


## 認証されたユーザーとして振舞う {#actions}

ユーザー認証用のトークンがあれば、トークンのスコープの範囲内で、そのユーザーとしてアクションを実行できるようになります。

### 投稿と投稿の削除 {#statuses}

* 投稿するには [POST /api/v1/statuses](../methods/statuses/#publish-new-status) を参照してください
  * 画像や動画を添付するには [/api/v1/media]({{< relref "../methods/statuses/media.md" >}}) を参照してください
  * スケジュールされた投稿をするには [/api/v1/scheduled\_statuses]({{< relref "../methods/statuses/scheduled_statuses.md" >}}) を参照してください

### タイムラインへの作用 {#timelines}

* タイムラインにアクセスするには [/api/v1/timelines]({{< relref "../methods/timelines/" >}}) を参照してください
* タイムラインの位置を保存したり読み出したりするには [/api/v1/markers]({{< relref "../methods/timelines/markers.md" >}}) を参照してください
* 投稿に対してアクションをするには [/api/v1/statuses]({{< relref "../methods/statuses/" >}}) を参照してください
  * 投票を見たり、投票を行うには [/api/v1/polls]({{< relref "../methods/statuses/polls.md" >}}) を参照してください
* [GET /api/v1/timelines/list/:list\_id](../methods/timelines/#list-timeline)でリストを取得するためのリストIDを取得するには [/api/v1/lists]({{< relref "../methods/timelines/lists.md" >}})  を参照してください
* ダイレクトメッセージを取得するには [/api/v1/conversations]({{< relref "../methods/timelines/conversations.md" >}}) を参照してください
* お気に入りのリスト表示をするには [/api/v1/favourites]({{< relref "../methods/accounts/favourites.md" >}}) を参照してください
* ブックマークのリスト表示をするには [/api/v1/bookmarks]({{< relref "../methods/accounts/bookmarks.md" >}}) を参照してください

### 他のユーザーへの作用 {#accounts}

* 他のユーザーに対してアクションを起こすには [/api/v1/accounts]({{< relref "../methods/accounts/" >}})  を参照してください
* フォローリクエストの管理をするには [/api/v1/follow\_requests]({{< relref "../methods/accounts/follow_requests.md" >}}) を参照してください
* ミュートのリスト表示するには [/api/v1/mutes]({{< relref "../methods/accounts/mutes.md" >}}) を参照してください
* ブロックのリスト表示するには [/api/v1/blocks]({{< relref "../methods/accounts/blocks.md" >}}) を参照してください

### 通知を受け取る {#notifications}

* ユーザーの通知を管理をするには [/api/v1/notifications]({{< relref "../methods/notifications/" >}}) を参照してください
* プッシュ通知を管理するには [/api/v1/push]({{< relref "../methods/notifications/push.md" >}}) を参照してください

### ディスカバリーを使う {#discovery}

* リソースの検索は [/api/v2/search]({{< relref "../methods/search.md" >}}) を参照してください
* おすすめアカウントの表示は [/api/v1/suggestions]({{< relref "../methods/accounts/suggestions.md" >}}) を参照してください

### 安全・安心機能を使う {#safety}

* キーワードフィルタリングを管理するには [/api/v1/filters]({{< relref "../methods/accounts/filters.md" >}}) を参照してください
* ドメインブロックを管理するには [/api/v1/domain\_blocks]({{< relref "../methods/accounts/domain_blocks.md" >}}) を参照してください
* 通報するには [/api/v1/reports]({{< relref "../methods/accounts/reports.md" >}}) を参照してください
* モデレーションは [/api/v1/admin]({{< relref "../methods/admin.md" >}}) を参照してください

### アカウント情報の管理 {#manage}

* プロフィールを管理するには [/api/v1/endorsements]({{< relref "../methods/accounts/endorsements.md" >}}) を参照してください
* おすすめハッシュタグを管理するには [/api/v1/featured\_tags]({{< relref "../methods/accounts/featured_tags.md" >}}) を参照してください
* ユーザー設定を取得するには [/api/v1/preferences]({{< relref "../methods/accounts/preferences.md" >}}) を参照してください

