---
title: ルート
description: How HTTP methods map to controllers and actions.
menu:
  docs:
    weight: 40
    parent: dev
---

{{< caption-link url="https://github.com/tootsuite/mastodon/blob/master/config/routes.rb" caption="config/routes.rb" >}}

## ルートの説明 {#routes}

MastodonはRuby on Rails を使用しており、config/routes.rb においてルーター設定が定義されています。[Ruby on Rails ルーティングガイド](https://guides.rubyonrails.org/routing.html) に詳細な情報があります。このページではMastodonがどのようにルーティングをするかの基本を解説します。

### どのようにルートが構築されているか {#router}

`namespace` は特定のコントローラーディレクトリに紐づけられたルートのプレフィックスです。`resources` は名前空間ディレクトリ内のコントローラーに紐づけられます。`scope` は `module` のコントローラーに渡されます。たとえば、次の簡略化されたコードについて見てみます：

{{< code title="config/routes.rb excerpt" >}}
```ruby
namespace :api do
    namespace :v1 do
        resources :statuses, only [:create, :show, :destroy] do
            scope module: :statuses do
                resource :favourite, only: :create
                resource :unfavourite, to: 'favourites#destroy'
                member do
                    get :context
                end
            end
        end
    end
end
```
{{< /code >}}

はじめに、使用できるリソースは、:api と :v1 名前空間の下にネストされた :statusesです。したがって、HTTPルートは /api/v1/statuses になります。`only` は、`app/controllers/api/v1/statuses_controller.rb`のコントローラーで定義された特定の許可されたメソッドを定義します。

/api/v1/statuses 内には、追加のリソースが定義されている :statuses のスコープがあります。これらのリソースのコントローラーは `app/controllers/api/v1/statuses/` にあります。たとえば、 :favourite は `app/controllers/api/v1/statuses/favourites_controller.rb` の \#create アクションで取り扱われます。:unfavourite も同じコントローラーで取り扱われますが、 \#destroy アクションによってです。

このスコープ内の `member` に対して定義されたカスタムメソッドもあります。つまり、statusが`app/controllers/api/v1/statuses_controller.rb`で制御され、/api/v1/statuses/:id/contextにマップされ、そのコントローラー内で定義された :context アクションによって処理されます。


### 使用可能メソッド {#methods}

#### :index

HTTP GETにマッピングされ、リストに使われます。コントローラーの \#index アクションでハンドルされます。

#### :show

HTTP GETにマッピングされ、単一ビューに使われます。コントローラーの \#show アクションでハンドルされます。

#### :create

HTTP POSTにマッピングされます。コントローラーの \#create アクションでハンドルされます。

#### :update

HTTP PUTにマッピングされます。コントローラーの \#update アクションでハンドルされます。

#### :destroy

HTTP DELETEにマッピングされます。コントローラーの \#destroy アクションでハンドルされます。

## .well-known {#well-known}

### /.well-known/host-meta {#host-meta}

Extensible Resource Descriptor \(XRD\)。 Webfingerの存在を知らせます。

### /.well-known/nodeinfo {#nodeinfo}

 `/nodeinfo/2.0` にある Nodeinfo 2.0 エンドポイントにマッピングされており、ソフトウェアの名前とバージョン、プロトコル、統計、登録可能かどうかの通知に使用されます。

### /.well-know/webfinger {#webfinger}

ActivityPubのactor idを検出するために使われます。詳しくは [Spec compliance &gt; WebFinger]({{< relref "../spec/webfinger.md" >}}) を参照してください。

### /.well-known/change-password {#change-password}

アカウントの設定ページへマッピングされています。

### /.well-known/keybase-proof-config {#keybase}

Keybaseとの統合に使用され、どのユーザ名が受け入れられ、どこで証明がチェックされるかを定義します。

{{< hint style="warning" >}}
The sections below this point are under construction.
{{< /hint >}}

## Public URIs {#public}

* `/users/username` = ユーザー URI
* `/users/username/remote_follow` = リモートフォローダイアログ
* `/users/username/statuses/id` = 投稿 URI
* `/@username` = "トゥート" tab
* `/@username/with_replies` = "トゥートと返信" tab
* `/@username/media` = "メディア" tab
* `/@username/tagged/:hashtag` = tagged statuses by user
* `/@username/:status_id` = status permalink
* `/@username/:status_id/embed` = embeddable version
* `/interact/:status_id` = remote interaction dialog
* `/explore` = profile directory
* `/explore/:hashtag` = profiles with this hashtag in bio
* `/public` = 公開タイムラインのプレビュー
* `/about` = ランディングページ
* `/about/more` = 詳細情報
* `/terms` = 利用規約

## API {#api}

* /api/oembed
* /api/proofs
* /api/v1
  * [statuses]({{< relref "../methods/statuses/" >}}) \[create, show, destroy\]
    * reblogged\_by \[index\]
    * favourited\_by \[index\]
    * reblog \[create\]
    * unreblog \[POST reblog\#destroy\]
    * favourite \[create\]
    * unfavourite \[POST favourites\#destroy\]
    * bookmark \[create\]
    * unbookmark \[POST bookmarks\#destroy\]
    * mute \[create\]
    * unmute \[POST mutes\#destroy\]
    * pin \[create\]
    * unpin \[POST pins\#destroy\]
    * context \[GET\]
  * [timelines]({{< relref "../methods/timelines/" >}})
    * home \[show\]
    * public \[show\]
    * tag \[show\]
    * list \[show\]
  * [streaming]({{< relref "../methods/timelines/streaming.md" >}}) \[index\]
  * [custom\_emojis]({{< relref "../methods/instance/custom_emojis.md" >}}) \[index\]
  * [suggestions]({{< relref "../methods/accounts/suggestions.md" >}}) \[index, destroy\]
  * [scheduled\_statuses]({{< relref "../methods/statuses/scheduled_statuses.md" >}}) \[index, show, update, destroy\]
  * [preferences]({{< relref "../methods/accounts/preferences.md" >}}) \[index\]
  * [conversations]({{< relref "../methods/timelines/conversations.md" >}}) \[index, destroy\]
    * read \[POST\]
  * [media]({{< relref "../methods/statuses/media.md" >}}) \[create, update\]
  * [blocks]({{< relref "../methods/accounts/blocks.md" >}}) \[index\]
  * [mutes]({{< relref "../methods/accounts/mutes.md" >}}) \[index\]
  * [favourites]({{< relref "../methods/accounts/favourites.md" >}}) \[index\]
  * [bookmarks]({{< relref "../methods/accounts/bookmarks.md" >}}) \[index\]
  * [reports]({{< relref "../methods/accounts/reports.md" >}}) \[create\]
  * [trends]({{< relref "../methods/instance/trends.md" >}}) \[index\]
  * [filters]({{< relref "../methods/accounts/filters.md" >}}) \[index, create, show, update, destroy\]
  * [endorsements]({{< relref "../methods/accounts/endorsements.md" >}}) \[index\]
  * [markers]({{< relref "../methods/timelines/markers.md" >}}) \[index, create\]
  * [apps]({{< relref "../methods/apps/" >}}) \[create\]
    * verify\_credentials \[credentials\#show\]
  * [instance]({{< relref "../methods/instance/" >}}) \[show\]
    * peers \[index\]
    * activity \[show\]
  * [domain\_blocks]({{< relref "../methods/accounts/domain_blocks.md" >}}) \[show, create, destroy\]
  * [directory]({{< relref "../methods/instance/directory.md" >}}) \[show\]
  * [follow\_requests]({{< relref "../methods/accounts/follow_requests.md" >}}) \[index\]
    * authorize \[POST\]
    * reject \[POST\]
  * [notifications]({{< relref "../methods/notifications/" >}}) \[index, show\]
    * clear \[POST\]
    * dismiss \[POST\]
  * [accounts]({{< relref "../methods/accounts/" >}})
    * verify\_credentials \[GET credentials\#show\]
    * update\_credentials \[PATCH credentials\#update\]
    * search \[show \(search\#index\)\]
    * relationships \[index\]
  * [accounts]({{< relref "../methods/accounts/" >}}) \[create, show\]
    * statuses \[index accounts/statuses\]
    * followers \[index accounts/follower\_accounts\]
    * following \[index accounts/following\_accounts\]
    * lists \[index accounts/lists\]
    * identity\_proofs \[index accounts/identity\_proofs\]
    * follow \[POST\]
    * unfollow \[POST\]
    * block \[POST\]
    * unblock \[POST\]
    * mute \[POST\]
    * unmute \[POST\]
    * pin \[POST\]
    * unpin \[POST\]
  * [lists]({{< relref "../methods/timelines/lists.md" >}}) \[index, create, show, update, destroy\]
    * accounts \[POST accounts/pins\#destroy\]
  * [featured\_tags]({{< relref "../methods/accounts/featured_tags.md" >}}) \[index, create, destroy\]
    * suggestions \[GET suggestions\#index\]
  * [polls]({{< relref "../methods/statuses/polls.md" >}}) \[create, show\]
    * votes \[create polls/votes\]
  * [push]({{< relref "../methods/notifications/push.md" >}})
    * subscription \[create, show, update, destroy\]
  * [admin]({{< relref "../methods/admin.md" >}})
    * accounts \[index, show\]
      * enable \[POST\]
      * unsilence \[POST\]
      * unsuspend \[POST\]
      * approve \[POST\]
      * reject \[POST\]
      * action \[create account\_actions\]
    * reports \[index, show\]
      * assign\_to\_self \[POST\]
      * unassign \[POST\]
      * reopen \[POST\]
      * resolve \[POST\]
* /api/v2
  * [search]({{< relref "../methods/search.md" >}}) \[GET search\#index\]

