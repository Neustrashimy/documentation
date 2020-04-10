---
title: プログラムの構造
description: Where to find certain parts of the codebase.
menu:
  docs:
    weight: 30
    parent: dev
---

{{< hint style="danger" >}}
このページは工事中です！
{{< /hint >}}

### プログラムの構造 {#structure}

以下に示す概要は、完全または信頼できるものではなく、アプリケーションの概略を掴むために役立つおおまかなガイドとして見なしてください。

#### Ruby {#ruby}

| Path | Description |
| :--- | :--- |
| `app/controllers` | ビジネスロジックとテンプレートを結びつけるコード |
| `app/helpers` | ビューで使われるコード。たとえば共通の操作など。 |
| `app/lib` | ほかのカテゴリーに属さないコード |
| `app/models` | データエンティティを表すコード |
| `app/serializers` | モデルからJSONを生成するコード |
| `app/services` | 複数のモデルが関係する複雑なロジックの集合体 |
| `app/views` | HTMLやその他の出力のためのテンプレート |
| `app/workers` | リクエスト・レスポンス以外で実行されるコード |
| `spec` | 自動化されたテスト一覧 |

#### JavaScript {#javascript}

| Path | Description |
| :--- | :--- |
| `app/javascript/mastodon` | マルチカラム React.js アプリのコード |
| `app/javascript/packs` | React.jsを使っていないページのコード |

#### CSS とその他のアセット {#assets}

| Path | Description |
| :--- | :--- |
| `app/javascript/images` | 画像 |
| `app/javascript/styles` | SassからCSSに変換されたコード |

#### ローカライゼーション {#localizations}

| Path | Description |
| :--- | :--- |
| `config/locales` | サーバー側のYMLフォーマットローカライゼーションデータ |
| `app/javascript/mastodon/locales` | クライアント側のJSONフォーマットローカライゼーションデータ |

### ローカライゼーションメンテナンス {#localization-maintenance}

すべてのロケールファイルは正規化されており、フォーマットとキーの順序に一貫性があるため、バージョン管理の変更セットが最小限で済みます。

| Command | Description |
| :--- | :--- |
| `i18n-tasks normalize` | 正規化されたサーバ側の翻訳 |
| `yarn run manage:translations` | 正規化されたクライアント側の翻訳 |

