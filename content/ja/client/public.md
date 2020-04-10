---
title: 公開データで遊ぶ
description: Familiarizing yourself with endpoints and entities.
menu:
  docs:
    weight: 20
    parent: client
---

cURLやプログラミング言語のHTTPユーティリティもしくはライブラリを使用して、HTTPリクエストを生成する方法を理解したので、次に、エンドポイントとレスポンスについて学習します。

## エンドポイントの説明 {#endpoints}

すべてのHTTPリクエストは、ターゲットURLに対して行われます。ウェブサイトに対してデータを要求するときは、特定のURLを使用します。URLに応じて、HTTPサーバーによってリクエストが解釈され、適切なレスポンスが返されます。

例では `https://mastodon.example` でホストされている架空のMastodonサーバー mastodon.example を使用します。このWebサイトのルートは`/`であり、サブディレクトリとパスはエンドポイントと呼ばれます。Mastodonのエンドポイントは `/api` 名前空間の下にネストされ、現在はほとんどのメソッドが `/api/v1` の下にあります。リクエストはHTTPメソッドとエンドポイントごとにリストされます。たとえば、GET /api/v1/endpoint は、ドメインのそのエンドポイントに対して発行されたGETリクエスト、言い換えると`https://mastodon.example/api/v1/endpoint`として解釈されます。


## 公開タイムラインの取得 {#timelines}

Mastodonの公開データの最も基本的なもののひとつである、公開タイムラインを見てみましょう。

次のように、[GET /api/v1/timelines/public](../methods/timelines/#public-timeline) へリクエストを試みることができます：

```bash
curl https://mastodon.example/api/v1/timelines/public
```

おおっと！大量のテキストが返ってきました！ 公開タイムラインは、デフォルトで20件の投稿を返します。`limit`パラメーターを使用して、数を制限できます。ふたたび同じエンドポイントにリクエストを送ってみましょう。ただし今度は limit を 2に設定します：

```bash
curl https://mastodon.example/api/v1/timelines/public?limit=2
```

今回のレスポンスはより見やすいはずです。ツールを使用して、このJSONを整形できます。レスポンスは以下のようになります：


```javascript
[
  {
    "id": "103206804533200177",
    "created_at": "2019-11-26T23:27:31.000Z",
    ...
    "visibility": "public",
    ...
  },
  {
    "id": "103206804086086361",
    "created_at": "2019-11-26T23:27:32.000Z",
    ...
    "visibility": "public",
    ...
  }
]
```

同様に、 [GET /api/v1/timelines/tag/:hashtag](../methods/timelines/#hashtag-timeline) で、ハッシュタグを検索できます。ここで、コロンはエンドポイントとパスパラメーターの区切りであることを示します。 :hashtag の場合、ハッシュタグの名前を使うことを意味します（ここでも2件に制限します）：

```bash
curl https://mastodon.example/api/v1/timelines/tag/cats?limit=2
```

再度、2件のデータが [投稿]({{< relref "../entities/status.md" >}}) エンティティのJSON配列形式で返されました。JSONを配列で解析し、次にオブジェクトで解析します。Pythonを使っている場合、コードは以下のようになります：

```python
import requests
import json

response = requests.get("https://mastodon.example/api/v1/timelines/tag/cats?limit=2")
statuses = json.dumps(response.text) # これはJSONをPythonの辞書型のリストに変換します
assert statuses[0]["visibility"] == "public" # 公開タイムラインを読みたいです
print(statuses[0]["content"]) # 投稿の本文を表示します
```

{{< hint style="info" >}}
JSONを解析してプログラムで使用することは、プログラミング言語やプログラムデザインによって異なるので、このチュートリアルでは扱いません。それぞれのプログラミング言語でのJSONの取扱いについては、他のチュートリアルを参照してください。
{{< /hint >}}

{{< hint style="info" >}}
[MastoVue](https://mastovue.glitch.me) は、公開タイムラインを見ることができるアプリケーションの例です。
{{< /hint >}}

## 公開アカウントと投稿を取得する{#toots}

リクエストの作成方法とレスポンスの処理方法を理解したので、さらに多くの公開データを試すことができます。次のメソッドが興味深いかもしれません：

* アカウントのIDを知っている場合は、[GET /api/v1/accounts/:id](../methods/accounts/#account) を使うと [Account]({{< relref "../entities/account.md" >}}) エンティティを取得できます。
  * そのアカウントによる公開投稿を見る場合は、 [GET /api/v1/accounts/:id/statuses](../methods/accounts/#statuses) を使います。結果は [Status]({{< relref "../entities/status.md" >}}) エンティティの配列です。
* 投稿のIDを知っている場合は、 [GET /api/v1/statuses/:id](../methods/statuses/#view-specific-status) を使うとStatus エンティティを見ることができます。
  * さらに [GET /api/v1/statuses/:id/reblogged\_by](../methods/statuses/#boosted-by) を使うと、誰が投稿をブーストしたか見ることができます。
  * あるいは [GET /api/v1/statuses/:id/favourited\_by](../methods/statuses/#favourited-by) を使うと、誰がお気に入りしたかを見ることができます。
  * [GET /api/v1/statuses/:id/context](../methods/statuses/#parent-and-child-statuses) をリクエストすると、スレッドツリーの祖先と子孫が表示されます。
  * 投稿にアンケートが付加されていたのなら、 [GET /api/v1/polls/:id](../methods/statuses/polls.md#view-a-poll) でアンケートを個別に見ることができます。

アカウントと投稿のIDはMastodon Webサイトのデータベースに対してローカルであり、Mastodon Webサイトごとに異なります。

## 公開インスタンスデータを取得する {#instance}

最後に、Mastodon Webサイトの情報を匿名でリクエストすることができます。

* 一般的な情報は [GET /api/v1/instance](../methods/instance/#fetch-instance) で取得できます。
  * ピアリストは [GET /api/v1/instance/peers](../methods/instance/#list-of-connected-domains) で取得できます。
  * 週間アクティビティは [GET /api/v1/instance/activity](../methods/instance/#weekly-activity) で取得できます。
  * 使用可能な全てのカスタム絵文字は [GET /api/v1/custom\_emojis](../methods/instance/custom_emojis.md#custom-emoji) で取得できます。
* [GET /api/v1/directory](../methods/instance/directory.md#view-profile-directory) で、ディレクトリに掲載されている使用可能なすべてのプロフィールを取得できます。
* [GET /api/v1/trends](../methods/instance/trends.md#trending-tags) を見れば、現在のハッシュタグのトレンドを知ることができます。

{{< hint style="info" >}}
インスタンスのデータだけでできることの実際の例として、[emojos.in](https://emojos.in/)があります。これは指定したインスタンスの全てのカスタム絵文字をプレビューできるサービスです。
{{< /hint >}}

