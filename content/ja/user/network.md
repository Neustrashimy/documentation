---
title: ネットワーク機能を使う
description: Follow and talk to anyone from any server.
menu:
  docs:
    weight: 40
    parent: user
---

## 公開タイムラインを通してコンテンツを見る {#timelines}

{{< figure src="/assets/image%20%2830%29.png" caption="Posts within a public timeline" >}}

あなたが興味がありそうなコンテンツを見つけるために、Mastodonはすべての公開された投稿を見る方法を提供しています。しかしながら、全てのサーバー間でグローバルに共有されているわけではないので、公開されている _全て_ の投稿を見ることができるわけではありません。**連合タイムライン** を閲覧することで、あなたがいるサーバーが知っている限りの全ての公開されている投稿を見ることができます。サーバーが投稿を発券する方法はいろいろありますが、その大部分はあなたのサーバーの他のユーザーがフォローしている人からの投稿です。

連合タイムラインから、あなたがいるサーバー上で作成された公開投稿のみをフィルタリングして見るには、**ローカルタイムライン** を閲覧します。ここで言う「ローカル」とはサーバーを意味し、地理的な場所を指すものではないことに注意してください。

## 他の人の投稿とのやりとり {#actions}

{{< figure src="/assets/image%20%2821%29.png" caption="An expanded view can be loaded by clicking a status in the timeline." >}}

タイムラインから直接クイックアクションを実行することもできますし、投稿をクリックすることで正確なタイムスタンプ、やりとりの数、スレッド化された返信がある場合はその返信の数などの追加情報を表示する拡張ビューを表示することもできます。投稿に対して、以下のアクションを実行することができます：


* 矢印アイコンをクリックすることで **返信** することができます。あなたの投稿は、返信した投稿の下にスレッド表示されます。
* 回転している矢印アイコンをクリックすることで **ブースト** することができます。投稿があなたのプロフィール上で再共有されます。
* 星のアイコンをクリックすることで **お気に入り** することができます。投稿はお気に入りリストに追加され、投稿者にお気に入りされたことを通知します。
* リボンアイコンをクリックすることで **ブックマーク** することができます。通知されず、プライベートにブックマークリストに追加されます。
* 省略記号（・・・）アイコンをクリックすることで、追加オプションのある **メニュー** を開くことができます。

## 通知 {#notifications}

{{< figure src="/assets/image%20%2850%29.png" caption="Notifications column" >}}

他の人があなたの投稿に対してやりとりを行うと、その種類によって通知を受信します。通知カラムではすべての通知を一度に見ることができるほか、以下の種類によってフィルタリングすることができます：

* **返信：** あなたの投稿に誰かが返信したときに受け取ります。
* **お気に入り:** あなたの投稿を誰かがお気に入りしたときに受け取ります。
* **ブースト:** あなたの投稿を誰かがブーストしたときに受け取ります。
* **アンケート:** あなたが作ったり投票したアンケートの結果が出たときに受け取ります。
* **フォロー:** 誰かがあなたをフォローしたときに受け取ります。

## Following profiles {#follow}

![](/assets/image%20%2811%29.png)

As long as you encounter a person within your app’s user interface, e.g. the web interface on your home server, or your mobile app, you can just click “follow” and you won’t notice a difference if that person is on your server or not.

However if you come across someone’s public profile hosted on a different server, there’s an obstacle: That server sees you as just another anonymous visitor. Not to worry! You can simply copy the URL of that profile, or of one of their posts, and then paste that URL into the search function.

If you are visiting a public page on another Mastodon site, see [Using Mastodon outside of your site](external.md#remote-interactions-on-another-mastodon-site).

## Search {#search}

{{< figure src="/assets/image%20%2819%29.png" caption="The search function can be accessed from the sidebar." >}}

Mastodon's basic search allows logged-in users to find toots containing a specific hashtag, or to load a user or status directly if they know the URL or address. Searching for a term will show profiles whose username or display name contains that term, as well as hashtags that match or contain that term.

{{< figure src="/assets/image%20%2839%29.png" caption="An example of a toot being loaded directly by its URL." >}}

{{< figure src="/assets/image%20%2823%29.png" caption="An example of accounts returned when searching for &quot;cats&quot;." >}}

{{< figure src="/assets/image%20%2827%29.png" caption="An example of hashtags returned when searching for &quot;cats&quot;." >}}

Admins may optionally install full-text search. Mastodon’s full-text search allows logged-in users to find results from their own toots, their favourites, and their mentions. It deliberately does not allow searching for arbitrary strings in the entire database, in order to reduce the risk of abuse by people searching for controversial terms to find people to dogpile.

The following operators are supported:

* **"exact phrases"** will try to find the term inside the quote marks. This allows looking only for direct matches, such as `"look at my cluckers"` to find posts explicitly telling you to look at someone's cluckers.
* **-exclude** will exclude the term prepended by a minus sign. This allows filtering out certain terms, such as `animals -cats` to find posts about animals without posts about cats.
* **+include** will include the term after the plus sign. This allows searching for multiple terms that must be included, such as `cat +dog` to find posts about both cats and dogs.

## Direct conversations {#direct}

{{< figure src="/assets/image%20%2812%29.png" caption="A list of conversations containing direct messages." >}}

In Mastodon, direct messages are simply toots that have the "direct" visibility selected. Visibility can be selected per-post, which allows changing the privacy level later in a thread. The direct messages column currently shows a list of all conversations containing a direct post. Clicking on a conversation will load the associated thread.

{{< figure src="/assets/image%20%2857%29.png" caption="A direct message in a thread." >}}

## List timelines {#lists}

Lists are subsets of your home timeline. You can create a list, give it a name, and add users that you follow to that list.

![](/assets/image%20%2828%29.png)

Opening a list will load that list's timeline. List timelines contain only posts by members of that list, as well as replies to you or to other members of the list.

{{< figure src="/assets/image%20%285%29.png" caption="A list timeline" >}}

