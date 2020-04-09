---
title: 新しいインスタンスをセットアップする
description: Things to do after installing Mastodon
menu:
  docs:
    weight: 50
    parent: admin
---

## 管理者アカウントを作成する {#admin}

### ブラウザから {#admin-gui}

ブラウザから登録したあと、コマンドラインから新しく作ったアカウントに管理者特権を与えます。
ユーザー名が`alice`の場合は、下記の通りにします：


```bash
RAILS_ENV=production bin/tootctl accounts modify alice --role admin
```

### コマンドラインから {#admin-cli}

コマンドラインから新しいアカウントを作成することができます。

```bash
RAILS_ENV=production bin/tootctl accounts create \
  alice \
  --email alice@example.com \
  --confirmed \
  --role admin
```

ランダムに生成されたパスワードがターミナルに表示されます。

## サーバー情報を設定する {#info}

ログインしたら、**サイト設定**ページに移動します。 この情報を設定するのに技術的なことは必要ありませんが、人のためのサーバーを運用するためには重要な情報です。


| Setting | Meaning |
| :--- | :--- |
| 連絡先ユーザー名 | 誰がサーバーを管理しているのか知らせるため、あなたのユーザー名を記入します |
| ビジネスメールアドレス | アカウントを持っていない、ログインできなくなったユーザーがあなたに連絡を取るためのメールアドレスです |
| 短いサーバーの説明 | どうしてサーバーを動かそうとおもいましたか？誰のためのものですか？他との違いは？ |
| サーバーの説明 | ここには様々な情報を記入することができますが、行動規範を記入することをおすすめします。 |

入力し終わったら、"変更を保存"ボタンを押します。

