---
title: モデレート
description: Actions that can be taken against unwanted users or domains.
menu:
  docs:
    weight: 110
    parent: admin
---


## 個別モデレーション {#individual-moderation}

Mastodonのモデレーションは常にローカルに適用されます。つまりサーバーごとに行われるということです。あるサーバーの管理者・モデレーターは他のサーバーに影響を及ぼすことはできず、自らのサーバーにコピーされたデータのみに干渉できます。


### ログインの無効化 {#disable-login}

Mastodonアカウントは無効化することができます。これにより、無効になったユーザーは何もできなくなりますが、投稿はそのまま残されます。この制限はいつでも元に戻すことができます。また、これはあなたのサーバーに属しているユーザーにのみ適用できます。


### サイレンス {#silence-user}

Mastodonのサイレンスはサンドボックスと同義です。サイレンスされたアカウントはフォローされていないユーザーから隠されます。投稿は全て残されており、検索したり、メンションしたり、フォローしたりすることができますが、投稿はすべて不可視になります。

現時点において、サイレンスはフェデレーションに影響しません。ローカルでサイレンスされたアカウントは、ほかのサーバーでは _自動的にサイレンスされません_ 。

この制限はいつでも元に戻すことができます。


### サスペンド {#suspend-user}

Mastodonのサスペンドは削除と同義です。サスペンドされたアカウントは検索に現れませんし、プロフィールページは見えなくなり、全ての投稿・アップロード・フォロワーやその他のデータは削除されます。この制限は **元に戻すことができません** 。サスペンドから戻してアクセス権をユーザーに返すことはできますが、古いデータは削除されています。


## サーバーごとのモデレーション {#server-wide-moderation}

不作法に振る舞うサーバーから大量のユーザーをモデレートするのは大変な作業になる可能性があります。特定のサーバーのすべてのユーザーに対し、先制的にモデレートすることが可能です。これを **ドメインブロック** と呼びます。これにはいくつかの制限レベルがあります。


### メディアファイルの拒否 {#reject-media}

このオプションが有効化されると、そのサーバーから転送されたファイル（アイコン、ヘッダー、絵文字、添付ファイルなど）はローカルで処理されなくなります。


### サイレンス {#silence-server}

そのサーバーの全てのアカウントをサイレンス状態にします。


### 停止 {#suspend-server}

そのサーバーのすべてのアカウントをサスペンド状態にします。そのサーバーからのデータは、ユーザー名を除いて、ローカルに保存されることはなくなります。


## スパム対策 {#spam-fighting-measures}

Mastodonにおいて、スパムを防止するためにいくつかの基本的な対策が施されています：

* 登録時にメールアドレスの確認が必要
* IPアドレスによる登録頻度の制限

しかし、熱心なスパマーはこれらを回避してきます。ほかの対応としては、**メールブラックリスト**という手段があります。登録時、MastodonはメールアドレスのAレコード、もしくはMXレコードの名前解決を行います。そして、IPアドレスをブラックリストと照合するのです。



### メールサーバーのブロック {#blocking-by-e-mail-server}

スパマーは、しばしば大量の異なるドメインのメールアドレスを用いて、個々のブラックリスト登録を困難にします。しかし、それらのドメインは単一のIPアドレスに紐づけられていることがあります。短期間に大量のスパム登録があった場合、DNS検索ツールやLinuxの`dig`ユーティリティを使ってチェックすることができます。たとえば、`dig 1.2.3.4`と実行すると、そのIPに紐づけられている全てのDNSレコードが表示されます。もし全てのドメインがそのIPアドレスに紐づいていることに気づいた場合、メールドメインブラックリストに登録すると効果的です。



### IPアドレスでブロックする {#blocking-by-ip}

Mastodonだけでは、訪問者をIPアドレスで制限することはできず、あまりよい手段とはいえません。なぜならIPアドレスは時折たくさんのユーザーと共有されていることがあり、場合により使用者が異なる場合があるためです。しかしLinuxのファイアウォールを使用すれば訪問者をIPアドレスで制限することができます。以下は`iptables`と`ipset`を使う場合の例です：


```bash
# ipsetのインストール
sudo apt install ipset
# "spambots" という名前のブラックリストを作成
sudo ipset create spambots nethash
# 1.2.3.4 をブラックリストに追加
sudo ipset add spambots 1.2.3.4
# ファイアウォールのルールにブラックリストを登録
sudo iptables -I INPUT 1 -m set --match-set spambots src -j DROP
```

自身の端末を拒否しないように気を付けてください！

