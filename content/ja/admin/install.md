---
title: ソースコードからインストール
description: Instructional guide on creating your own Mastodon-powered website.
menu:
  docs:
    weight: 20
    parent: admin
---

## 前提条件 {#pre-requisites}

* **Ubuntu 18.04** が動いており、rootアクセスができるサーバー。 
* Mastodonサーバー用の **ドメイン** \(もしくはサブドメイン\)。たとえば `example.com` のような。
* 電子メール配送サービス、もしくは他の **SMTP サーバー**

rootでコマンドを実行する必要があります。rootでない場合は、rootに切り替えます。


### System repositories {#system-repositories}

まずcurlがインストールされていることを確認します。

#### Node.js {#node-js}

```bash
curl -sL https://deb.nodesource.com/setup_12.x | bash -
```

#### Yarn {#yarn}

```bash
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
```

### System packages {#system-packages}

```bash
apt update
apt install -y \
  imagemagick ffmpeg libpq-dev libxml2-dev libxslt1-dev file git-core \
  g++ libprotobuf-dev protobuf-compiler pkg-config nodejs gcc autoconf \
  bison build-essential libssl-dev libyaml-dev libreadline6-dev \
  zlib1g-dev libncurses5-dev libffi-dev libgdbm5 libgdbm-dev \
  nginx redis-server redis-tools postgresql postgresql-contrib \
  certbot python-certbot-nginx yarn libidn11-dev libicu-dev libjemalloc-dev
```

### Rubyのインストール {#installing-ruby}

Rubyのバージョンを管理するために rbenv を使用します。なぜなら正しいバージョンを取得したり、新しいバージョンがリリースされた場合にアップデートすることが容易だからです。
rbenvは単一のLinuxユーザーに対してインストールされるべきです。そのため、最初にMastodonを動作させるためのユーザーを作成します：

```bash
adduser --disabled-login mastodon
```

Mastodon用のユーザーに切り替えます：

```bash
su - mastodon
```

続いて、rbenv と rbenv-build をインストールします：

```bash
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
cd ~/.rbenv && src/configure && make -C src
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec bash
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```

これが終わると、正しいRubyバージョンをインストールできるようになります：

```bash
RUBY_CONFIGURE_OPTS=--with-jemalloc rbenv install 2.6.6
rbenv global 2.6.6
```

bundlerのインストールも必要です：

```bash
gem install bundler --no-document
```

rootユーザーに戻ります：

```bash
exit
```

## Setup {#setup}

### PostgreSQLのセットアップ {#setting-up-postgresql}

#### パフォーマンス設定 \(オプション\) {#performance-configuration-optional}

最適なパフォーマンスのため、[pgTune](https://pgtune.leopard.in.ua/#/)を使って、PostgreSQLを`systemctl restart postgresql`コマンドで再起動する前に、適切な設定値を`/etc/postgresql/9.6/main/postgresql.conf`に書き込みます。


#### ユーザーの作成 {#creating-a-user}

Mastodonが使用するPostgreSQLユーザーを作る必要があります。これは"ident"認証を使うもっとも簡単なセットアップ方法です。これはPostgreSQLユーザーは個別のパスワードを持たず、Linuxユーザーと同じユーザー名を使用します。


psqlプロンプトを開きます:

```bash
sudo -u postgres psql
```

psqlプロンプトで、下記コマンドを実行します：

```sql
CREATE USER mastodon CREATEDB;
\q
```

終わりです！

### Mastodonの設定 {#setting-up-mastodon}

いよいよMastodonのプログラムをダウンロードする時です。Mastodonユーザーに切り替えます：

```bash
su - mastodon
```

#### プログラムのチェックアウト {#checking-out-the-code}

Gitを使用し、最新のMastodonの安定版を取得します：

```bash
git clone https://github.com/tootsuite/mastodon.git live && cd live
git checkout $(git tag -l | grep -v 'rc[0-9]*$' | sort -V | tail -n 1)
```

#### 依存性パッケージのインストール {#installing-the-last-dependencies}

RubyとJavaScriptの依存性パッケージをインストールします：

```bash
bundle config set deployment 'true'
bundle config set without 'development test'
bundle install -j$(getconf _NPROCESSORS_ONLN)
yarn install --pure-lockfile
```

{{< hint style="info" >}}
ふたつの`bundle config`コマンドは依存性パッケージを最初にインストールするときのみ必要です。あとからアップデートや依存性パッケージを再インストールするときは、`bundle install`とだけ実行すれば十分です。
{{< /hint >}}

#### 設定ファイル生成 {#generating-a-configuration}

対話的セットアップウィザードを起動します：

```bash
RAILS_ENV=production bundle exec rake mastodon:setup
```

これは、次のようなことをします:

* 設定ファイルを生成
* アセット（静的ファイル）の事前コンパイル
* データベーススキーマ（構造）の作成

設定ファイルは`.env.production`という名前で保存されます。自由に内容を見たり、好きなように編集することができます。[設定値のドキュメント]({{< relref "config.md" >}})を参照してください。

Mastodonユーザーでの作業は終わりです。rootユーザーに戻ります：

```bash
exit
```

### Nginxのセットアップ {#setting-up-nginx}

Nginx用の設定テンプレートをMastodonディレクトリからコピーします：

```bash
cp /home/mastodon/live/dist/nginx.conf /etc/nginx/sites-available/mastodon
ln -s /etc/nginx/sites-available/mastodon /etc/nginx/sites-enabled/mastodon
```

次に、`/etc/nginx/sites-available/mastodon`を編集し、`example.com`をあなたのドメイン名に置き換えます。他の設定値も必要に応じて調整します。

Nginxをリロードすると変更が有効になります：


### SSL証明書の取得 {#acquiring-a-ssl-certificate}

Let’s EncryptのSSL証明書を使用します：

```bash
certbot --nginx -d example.com
```

これで証明書を取得し、自動的に新しい証明書を使うよう`/etc/nginx/sites-available/mastodon`を更新します。Nginxをリロードすると、変更が有効になります。

この時点で、あなたのドメインにブラウザでアクセスすることができます。そして、象がコンピューターを叩いているエラー画面を見ることができます。これは、まだMastodonプロセスを動かしていないためです。

### systemdサービスのセットアップ {#setting-up-systemd-services}

systemdサービス用のテンプレートをMastodonディレクトリからコピーします：

```bash
cp /home/mastodon/live/dist/mastodon-*.service /etc/systemd/system/
```

続いて、次のファイルのユーザー名とパスを正しく編集します：

* `/etc/systemd/system/mastodon-web.service`
* `/etc/systemd/system/mastodon-sidekiq.service`
* `/etc/systemd/system/mastodon-streaming.service`

最後に、新しいsystemdサービスを実行し、自動起動するよう設定します：

```bash
systemctl daemon-reload
systemctl start mastodon-web mastodon-sidekiq mastodon-streaming
systemctl enable mastodon-*
```

それらはサーバーの起動時に自動的に開始されるようになります。


{{< hint style="success" >}}
**バンザーイ！ これで終わりです。あなたのドメインにブラウザでアクセスしてみてください！**
{{< /hint >}}

