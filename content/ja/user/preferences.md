---
title: 環境設定をする
description: Customize things just the way you like them.
menu:
  docs:
    weight: 70
    parent: user
---

## ユーザーインタフェイスのカスタマイズ {#interface}

### テーマを選ぶ {#theme}

Mastodonはデフォルトでダークテーマが適用されていますが、ライトテーマやハイコントラストテーマを選ぶことができます。

{{< figure src="/assets/image%20%2834%29.png" caption="Mastodon light theme" >}}

### レイアウトを選ぶ {#layout}

Mastodonはデフォルトでシンプルな1カラムレイアウトが適用されています。複数のカラムを同時に見たり、ピン止めできたりする高度なWebインタフェイスも選ぶことができます。

{{< figure src="/assets/image%20%2832%29.png" caption="The advanced web interface" >}}

どちらのインタフェイスでも、新しい投稿が利用可能であれば自動的に読み込みます。代わりに低速モードを使用すると、新しいアイテムが利用可能になったとき、カラム上部に数量のバナーを表示します。それらのアイテムはバナーをクリックしたときのみ読み込まれます。

アクセシビリティの観点から、アニメーションGIFの自動再生はデフォルトで無効になっています。アニメーションを見たい場合は、アニメーションGIFを有効にすることができます。また、ユーザーインタフェイスのアニメーションも抑制することができます。

トレンドタグは、高度なUIを使用しているときは「スタート」カラムの下部に、シンプルUIを使っている場合はカラムスイッチャーの下部に表示されます（表示するスペースがある場合）。


### 確認ダイアログ {#confirm}

いくつかのアクションを行う前に、確認を求めるかどうかを選ぶことができます。現在、以下のアクションを行う前に確認ダイアログを表示させることができます：

* フォロー解除
* ブースト
* 削除

### センシティブコンテンツ {#sensitive}

デフォルトでは、センシティブだとマークされたメディアはクリックすると外れるオーバレイで隠されます。センシティブとマークされているかに関わらず、常に隠す、もしくは常に隠さず表示するかを選ぶことができます。

隠され、読み込まれていないメディアは、BlurHashアルゴリズムにより、元の画像の色味を活かしたカラフルなぼかし表示されます。ぼかしは無効化できます。

{{< figure src="/assets/image%20%286%29.png" caption="An example blurhash thumbnail" >}}

コンテンツワーニングが付いた投稿はデフォルトで折り畳まれますが、常に展開して投稿全体を見るように設定することができます。

（訳注：センシティブコンテンツとは、日本語においては「閲覧注意」と訳されています。また、NSFW：職場閲覧注意 とも呼ばれます。）


## 通知の制御 {#notifications}

### メールの送信 {#email}

Mastodon内で受け取れる通知の種類に応じて、メール通知を行うか選ぶことができます。次のような通知で使用できます：

* フォローされた
* フォローリクエストを受けた
* ブーストされた
* お気に入りされた
* 返信された

長期間アクティブでなかった間に受け取った通知のダイジェストをメールで受け取るようにすることもできます。


### 特定の通知を隠す {#hide-notifications}

フォローしていない、もしくはフォローされていない人々からの通知を受け取らないようにすることもできます。返信、お気に入り、ブーストなどのやりとりについての通知が見えなくなります。

あなたがフォローしていない人々からのダイレクトメッセージの通知を受け取らないようにすることもできます。

## その他のオプション {#misc}

サーチエンジンからオプトアウトするために、公開プロフィールページと投稿ページに `noindex` フラグを付与することができます。

フォロー、フォロワーのリストを公開しないことで、あなたのつながりを隠すことができます。

{{< figure src="/assets/image%20%284%29.png" caption="A profile that has opted to hide its network" >}}

複数回ブーストされた投稿をフィードのトップに持ってきたい場合は、タイムラインでのブーストのグループ化を無効にすることができます。

### デフォルトの投稿の公開範囲 {#posting}

投稿はデフォルトで公開になっています。デフォルトは、非収載、フォロワー限定に切り替えることができます。公開範囲について詳しくは [トゥートを投稿する &gt; 公開範囲](posting.md#privacy) を参照してください。

デフォルトでは、あなたの投稿の言語は自動検出されます。しかしこの検出では正確でない可能性があります。主に、または専ら特定の言語で投稿する場合は、ここでその言語を設定することをおすすめします。

もしセンシティブなメディアを頻繁に投稿するのなら、常にメディアを閲覧注意にすることもできます。

### 公開タイムラインの言語フィルタリング {#languages}

公開タイムラインで、特定の言語でのみ書かれた投稿を表示するよう選択できます。ただし、言語検出はとても不正確であるため、無効にした言語での投稿が見えたり、有効にした言語の投稿を見逃したりする可能性があることに注意してください。