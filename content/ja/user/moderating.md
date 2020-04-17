---
title: 不要なコンテンツへの対処法
description: Control what you see, for a more comfortable social media experience.
menu:
  docs:
    weight: 50
    parent: user
---

## 投稿のフィルタリング {#filters}

投稿を特定のキーワードやフレーズでフィルタリングすることができ、それらは自動的に隠されます。

{{< figure src="/assets/image%20%2848%29.png" caption="A sample of active filters for various keywords in different contexts." >}}

フィルターを作成したり管理するには、設定 &gt; フィルター から行います。「新規フィルターを追加」ボタンを押すと、新しいフィルターを作ることができます。既存のフィルターは編集したり削除できます。既存のフィルターは表形式にまとめられます。

{{< figure src="/assets/image%20%2814%29.png" caption="Filters can have an expiry date, specific contexts, server-side drop, and use word boundaries." >}}

フィルターにはいくつか設定があります：

### キーワードまたはフレーズ {#filter-phrase}

一致させる文字列です。キーワードは、CWやメディアの説明文、アンケートのオプションを含む、すべての投稿の内容から探されます。

### 有効期限 {#filter-expire}

オプションですが、フィルターを適用する期間を設定できます。期限が切れたフィルターは自動的に削除されず、新しい有効期限を設定することで再度有効にできます（もしくは「無期限」に戻すこともできます）。

### 除外対象 {#filter-context}

フィルターを適用する対象を選べます：

* ホームタイムライン = 一致した投稿をホームフィードから除きます
* 通知 = 一致した通知が表示されなくなります
* 公開タイムライン = 一致した投稿がローカル・連合タイムラインに表示されなくなります
* 会話 = 一致した投稿はスレッドと詳細表示画面から隠されます

### 隠すのではなく除外する {#filter-drop}

フィルタリングは通常、クライアント側で行われます。そのため、フィルターを無効にするとフィルタリングされていた投稿を再び見ることができます。「隠すのではなく除外する」を有効にした場合、一致した投稿はホームフィードや通知に完全に表示されなくなります。

### 単語全体にマッチ {#filter-whole}

フィルターは通常、単語の位置に関わらず、その文字列を含むすべての投稿に適用されます。「単語全体にマッチ」を有効にすると、キーワードがスペースや英数字以外の文字で囲まれている場合にのみフィルターが適用されます。

## ユーザー側の操作 {#blocking-and-muting}

{{< figure src="/assets/image%20%2824%29.png" caption="The user dropdown menu offers various actions." >}}

### ブーストの非表示 {#hide-boosts}

誰かのブーストを非表示にした場合、彼らのブーストはあなたのホームフィードに表示されなくなります。このオプションはあなたがフォロー中のユーザーにのみ適用できます。

### ミュート {#mute}

{{< figure src="/assets/image%20%2852%29.png" caption="Sample of muted accounts." >}}

ミュートを行う際は、ミュートしたユーザーからの通知を受け取るかどうかを選ぶことができます。通知をミュートせずミュートを行った場合の動作は：

* そのユーザーはホームフィードでは見えません
* 誰かがそのユーザーの投稿をブーストしても見えません
* 誰かがそのユーザーに返信をしても見えません
* そのユーザーは公開タイムラインに現れません

もし通知をミュートする選択をした場合は、さらにそのユーザーからの通知を受け取らなくなります。

ユーザーには、自身がミュートされているかを知る術がありません。

### ブロック {#block}

{{< figure src="/assets/image%20%2836%29.png" caption="Sample of blocked accounts." >}}

ブロックすると、あなたの画面から以下のものが隠されます：

* そのユーザーはホームフィードでは見えません
* 誰かがそのユーザーの投稿をブーストしても見えません
* 誰かがそのユーザーに返信をしても見えません
* そのユーザーは公開タイムラインに現れません
* そのユーザーからの通知を受け取りません

さらに、ブロックされたユーザー側から見たとき：

* あなたを強制的にアンフォローします
* あなたをフォローできなくなります
* 誰かがあなたの投稿をブーストしても見えません
* あなたは公開タイムラインに現れません

もし、あなたとブロックしたユーザーが同じサーバーに属していたとき、ブロックされたユーザーからは、ログイン中はあなたの投稿やプロフィールを見ることができなくなります。

### Hiding an entire server {#hide-domain}

![](/assets/image%20%2861%29.png)

If you hide an entire server:

* You will not see posts from that server on the public timelines
* You won’t see other people’s boosts of that server in your home feed
* You won’t see notifications from that server
* You will lose any followers that you might have had on that server

## Reporting problematic content to moderators {#report}

{{< figure src="/assets/image%20%283%29.png" caption="The report modal allows selecting example statuses, adding a note, and forwarding reports." >}}

If you see a status or user that is violating the rules of your website, you can report that user to your site's moderators. Clicking the "report" option on the user dropdown or status dropdown will open the report modal. Here, you can \(and should\) add a note about why you are reporting this account. You can attach certain problematic statuses for additional context on why you are reporting the account, and if their conduct is violating the rules of the remote website, you can also choose to forward the report to their site's moderators.

