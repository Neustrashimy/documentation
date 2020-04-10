---
title: APIを使い始める
description: 'A primer on REST APIs, HTTP requests and responses, and parameters.'
menu:
  docs:
    weight: 10
    parent: client
---

## RESTの紹介 {#rest}

Mastodonはデータに対してREST API経由でのアクセスを提供します。RESTとは、REpresentational State Transferの略ですが、ここではリクエストに基づいてさまざまなリソースに関する情報を送受信するものだと考えてください。MastodonのREST APIはHTTPでリクエストされ、JSONで返されます。


## HTTPリクエストとレスポンスを理解する {#http}

REST APIエンドポイントは特定のHTTPメソッドで呼び出すことができ、同じエンドポイントで複数のメソッドを使用できます。Mastodon APIは、一般的に次のようなHTTPメソッドで呼び出されます：

* **GET**: リソースの読み込み、表示
* **POST**: サーバーへの情報送信
* **PUT** \| **PATCH**: リソースの更新
* **DELETE**: リソースの削除

お使いのプログラミング言語には、HTTPリクエストを行うためのユーティリティやライブラリがおそらくあります。このセクションでは、例としてcURLユーティリティを使用します。これは、多くのオペレーティングシステムにデフォルトで含まれているコマンドラインユーティリティです（`curl`として）。

cURLでは、デフォルトのHTTPメソッドはGETですが、`--request` もしくは `-X`フラグを用いることでリクエストの種類を指定することができます。たとえば、`curl -X POST`とすると、GETリクエストではなくPOSTリクエストを送信します。また、`-i`フラグを用いることで、追加のHTTPヘッダーを含めることができます。

## パラメーターを設定する {#parameters}

HTTPリクエストには、さまざまな方法で追加のパラメーターを含めることができますが、Mastodon APIは特にクエリー文字列、フォームデータ、JSONを解釈します。

{{< hint style="info" >}}
クエリー文字列、フォームデータ、POSTボディに含まれたJSONは、APIでは同様に解釈されます。GETリクエストにはクエリー文字列が使用され、その他すべてのリクエストにはフォームデータもしくはJSONが使用されることを想定しています。
{{< /hint >}}

### クエリー文字列 {#query-strings}

シンプルにURLを要求しますが、最後にクエリー文字列を追加します。クエリー文字列は、最初に ? を付け、続いて parameter=value の形式で追加します。& で区切ることで、複数のクエリー文字列を追加できます。たとえば：

```bash
curl https://mastodon.example/endpoint?q=test&n=0
```

### フォームデータ {#form-data}

クエリー文字列でURLを変更する代わりに、データを個別に送信できます。cURLでは、`--data` もしくは `-d` フラグを指定してデータを渡します。データは、クエリー文字列と同様に一緒に送信される場合と、複数のキーと値のペアとして個別に送信される場合があります。`--form` もしくは `-F` フラグを指定することで、キーと値のペアで送信できます。これにより、ファイルなどのマルチパートデータも送信できます。たとえば：

```bash
# send raw data as query strings
curl -X POST \
     -d 'q=test&n=0' \
     https://mastodon.example/endpoint
# send raw data separately
curl -X POST \
     -d 'q=test' \
     -d 'n=0' \
     https://mastodon.example/endpoint
# explicit form-encoded; allows for multipart data
curl -X POST \
     -F 'q=test' \
     -F 'n=0' \
     -F 'file=@filename.jpg' \
     https://mastodon.example/endpoint
```

### JSON {#json}

フォームデータを送信する時と似ていますが、データがJSON形式であることを示すヘッダーを追加する必要があります。cURLでJSONリクエストを送信するときは、Content Type として JSON を指定してから、フォームデータとしてJSONデータを送信します：

```bash
curl -X POST \
     -H 'Content-Type: application/json' \
     -d '{"parameter": "value}' \
     https://mastodon.example/endpoint
```

## データ型 {#types}

### 複数の値 \(Array\) {#array}

パラメータの配列はブラケット表記に変換しなければなりません。たとえば `array[]=foo&array[]=bar` は次のように変換されます：

```ruby
array = [
  'foo',
  'bar',
]
```

JSONでは次のように表現されます：

```javascript
{
  "array": ["foo", "bar"]
}
```

### ネストされたパラメータ \(Hash\) {#hash}

いくつかのパラメータはネストする必要があります。そのためには同じようにブラケット表示が使用されます。たとえば `source[privacy]=public&source[language]=en` は次のように変換されます：


```ruby
source = {
  privacy: 'public',
  language: 'en',
}
```

JSONでは次のように表現されます：

```javascript
{
  "source": {
    "privacy": "public",
    "language", "en"
  }
}
```

### 真偽値 \(ブーリアン型\) {#boolean}

ブール値は `0`, `f`, `F`, `false`, `FALSE`, `off`, `OFF` といった値をfalse（偽）と見なし、空白文字列は値がないものと見なし、他の値はtrue（真）であると見なされます。JSONデータで使う場合は、代わりに `true`、`false`そして`null`を使用してください。

### ファイル {#file}

ファイルをアップロードする場合は、`multipart/form-data`を使ってエンコードしてください。

これは配列と組み合わせることもできます。

## レスポンスデータの取扱い {#responses}

MastodonのREST APIはレスポンステキストとしてJSONを返します。また、HTTPヘッダーもレスポンスを取り扱うときには有用です。HTTPステータスコードによって、サーバーがどのようにリクエストを取り扱ったか知ることができます。以下のHTTPステータスコードが期待されます：


* 200 = OK. リクエストは正常に解釈された。
* 4xx = クライアントエラー。不正なリクエスト。一般的に、401 Unauthorized（認証が必要）、 404 Not Found（リソースが見つからない）、 410 Gone（リソースは恒久的に消滅した）、422 Unprocessed（処理できない） といったコードを見ることができます。
* 5xx = サーバーエラー。リクエスト処理中に何らかのおかしなことが起きた。一般的に、503 Unavailable（サービス利用不可） といったコードを見ることができます。

