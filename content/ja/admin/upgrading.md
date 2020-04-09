---
title: 新しいバージョンにアップグレードする
menu:
  docs:
    weight: 70
    parent: admin
---

{{< hint style="info" >}}
Mastodonの新しいバージョンがリリースされると、[GitHubのリリースページ](https://github.com/tootsuite/mastodon/releases)に現れます。`master`ブランチのリリース前コードで動かすことは可能ですが推奨されません。
{{< /hint >}}

MastodonのリリースはGitタグに対応しています。アップグレードを行う前に、目的のリリースについて、[GitHubのリリースページ](https://github.com/tootsuite/mastodon/releases)で調べます。このページには、何が新しくなったのか知ることのできる**changelog**が含まれるほか、**アップグレード手順**も含まれます。

始めるには、`mastodon`ユーザーに切り替えます：

```bash
su - mastodon
```

Mastodonのルートディレクトリまで移動します：

```bash
cd /home/mastodon/live
```

リリースされたコードをダウンロードします。下記は`v3.1.2`を指定した場合です：

```bash
git fetch --tags
git checkout v3.1.2
```

GitHubのリリースノートに掛かれている、そのバージョンのアップグレード手順に従ってアップグレードを実行します。リリースごとに異なる手順が求められるため、このページでは記載しません。


{{< hint style="info" >}}
古いバージョンからアップグレードする場合は、中間バージョンをスキップすることができます。個別にチェックアウトする必要はありません。ただし、リリースごとに手順を追う必要があります。ほとんどの手順は重複していますが、少なくとも1回は全てを実行する必要があります。
{{< /hint >}}

リリースノートに書かれた手順をすべて実行し終えたら、rootに戻ります：

```bash
exit
```

**バックグラウンドワーカー** を再起動します:

```bash
systemctl restart mastodon-sidekiq
```

そして**Webプロセス**を再読み込みします:

```bash
systemctl reload mastodon-web
```

{{< hint style="info" >}}
`reload`はダウンタイムなしで再起動することができ、"段階的再起動"とも呼ばれます。そのため、通常、Mastodonのアップグレードでは、計画的なダウンタイムについてユーザーに事前通知する必要はありません。`restart`も使用することができますが、ユーザーに対するサービスが（ごく短時間）中断されます。
{{< /hint >}}

稀ですが、**ストリーミング API** サーバーが更新された場合は再起動が必要になります：

```bash
systemctl restart mastodon-streaming
```

{{< hint style="danger" >}}
ストリーミングAPIサーバーが更新されることはほとんどなく、ほとんどのリリースでは再起動は *必要ありません* 。ストリーミングAPIを再起動すると、切断されたクライアントが代わりにREST APIに再接続もしくはポーリングしようとするため、サーバーの負荷が増加します。可能なら回避してください。
{{< /hint >}}

{{< hint style="success" >}}
**終わりです！** 新しいバージョンのMastodonが動いています。
{{< /hint >}}
