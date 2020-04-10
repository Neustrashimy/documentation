---
title: クライアントアプリアクセスの取得
description: Getting accustomed to the basics of authentication and authorization.
menu:
  docs:
    weight: 30
    parent: client
---

## 認証と承認 {#auth}

これまでは公開されている情報を使用してきましたが、すべての情報が公開されているわけではありません。一部の情報は、誰がその情報を要求したか監査するために、閲覧する前に許可が必要になります（そして許可の取消し、拒否をする可能性があります）。

ここで [OAuth]({{< relref "../spec/oauth.md" >}}) の出番です。OAuthは、リクエストが特定のクライアントからのものであることを _認証（Authenticate） （検証）_ し、リクエストされたアクションがサーバーのアクセス制御ポリシーによって _承認（Authorized）（許可）_ されることを保証するために使用できるアクセストークンを生成するメカニズムです。

## アプリを作る {#app}

最初にすることは、アクセストークンを生成できるようにするために、アプリを登録することです。次のようにします：

```bash
curl -X POST \
	-F 'client_name=Test Application' \
	-F 'redirect_uris=urn:ietf:wg:oauth:2.0:oob' \
	-F 'scopes=read write follow push' \
	-F 'website=https://myapp.example' \
	https://mastodon.example/api/v1/apps
```

上記の例では、クライアント名とWebサイトを指定しています。これは投稿に表示されます。ただし、さらに重要なこととして、次の2つのパラメーターに注意してください：

* `redirect_uris` は、"アウトオブバンド"のトークン生成に設定されています。これは、生成されたトークンを主導でコピー＆ペーストする必要があるということです。複数のリダイレクトURIを設定できるため、このパラメーターは`redirect_uris`と呼ばれます。トークンを生成するときに、このリストに含まれるURIを提供する必要があります。
* `scopes` を使用すると、後で要求できる権限を定義できます。ただし、後で要求されたスコープは、これらの登録済みのスコープのサブセットになる場合があります。詳しくは [OAuth Scopes](../api/oauth-scopes.md) を参照してください。

Applicatonエンティティが返されているはずですが、ここでは client\_id と client\_secret について考慮します。それらの値はアクセストークンの生成に使用されるため、一時的に保存しておく必要があります。アプリの登録について、詳しくは [POST /api/v1/apps](../methods/apps/#create-an-application) を参照してください。


## 認証コードフローの例 {#flow}

これでアプリの登録ができたので、リクエストをそのクライアントアプリとして認証するため、[POST /oauth/token](../methods/apps/oauth.md#obtain-a-token) を使用してアクセストークンを取得しましょう：

```bash
curl -X POST \
	-F 'client_id=your_client_id_here' \
	-F 'client_secret=your_client_secret_here' \
	-F 'redirect_uri=urn:ietf:wg:oauth:2.0:oob' \
	-F 'grant_type=client_credentials' \
	https://mastodon.example/oauth/token
```

次の点に注意してください：

* `client_id` と `client_secret` は、アプリを登録した時のレスポンスに記載されています。
* `redirect_uri` は、アプリ登録時に設定したURIのうち1つでなければなりません。
* `grant_type` の `client_credentials` を要求していますが、デフォルトでは `read` スコープが与えられています。

このメソッドの戻り値は [Token]({{< relref "../entities/token.md" >}}) エンティティです。`access_token`の値が必要となります。アクセストークンを取得したら、ローカルキャッシュに保存してください。これをリクエストで使う場合は、OAuthが必要なAPI（公開アクセスできないもの）を呼び出すとき、HTTPヘッダーに`Authorization: Bearer ...` を追加します。取得した認証情報の検証のため、[GET /api/v1/accounts/verify\_credentials](../methods/accounts/#verify-account-credentials) を呼び出します：


```bash
curl \
	-H 'Authorization: Bearer our_access_token_here' \
	https://mastodon.example/api/v1/apps/verify_credentials
```

トークンを取得し、正しくリクエストをフォーマットできていれば、 `source`パラメータが含まれた詳細な [Account]({{< relref "../entities/account.md" >}}) エンティティが返されます。

## 認証するとできること {#methods}

認証されたクライアントアプリでは、アカウントのリレーションシップを [GET /api/v1/accounts/:id/following](../methods/accounts/#following) と [GET /api/v1/accounts/:id/followers](../methods/accounts/#followers) で見ることができます。また、一部のインスタンスでは、他では公開である部分にも認証が必要である場合があるため、公開メソッドで遊んでいる時に認証エラーが発生した場合、それらを動作させることができるようになるはずです。
