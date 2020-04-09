---
title: サーバーを準備する
menu:
  docs:
    weight: 10
    parent: admin
---

新しいサーバーをセットアップする場合、まずサーバーをセキュアにすることを推奨します。以下は**Ubuntu 18.04**を実行している場合です：

## SSHログインにパスワード認証を使わない \(keys only\)

先に証明書を用いてサーバーにSSHログインができることを確認してください。さもなくば締め出されてしまいます。多くのホスティング業者は公開鍵のアップロードに対応しており、証明書を用いたrootログインを自動的に設定してくれます。

`/etc/ssh/sshd_config` を編集し、`PasswordAuthentication`という項目を探してください。その項目のコメントを外し、`no`に設定します。変更を有効にするために、sshdを再起動します：

```bash
systemctl restart sshd
```

## システムパッケージのアップデート

```bash
apt update && apt upgrade -y
```

## fail2ban をインストールして、複数回のログイン試行をブロック

`/etc/fail2ban/jail.local` を編集し、以下の設定を書き込みます：

```text
[DEFAULT]
destemail = your@email.here
sendername = Fail2Ban

[sshd]
enabled = true
port = 22

[sshd-ddos]
enabled = true
port = 22
```

最後にfail2banを再起動します:

```bash
systemctl restart fail2ban
```

## ファイアウォールをインストールし、SSHのホワイトリスト・HTTP・HTTPSポートの設定を行う

先に、iptables-persistentをインストールします。インストール時に、現在のルールセットを残すかどうか聞かれます。

```bash
apt install -y iptables-persistent
```

`/etc/iptables/rules.v4`を編集し、以下の設定を書き込みます：

```text
*filter

#  Allow all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT

#  Accept all established inbound connections
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#  Allow all outbound traffic - you can modify this to only allow certain traffic
-A OUTPUT -j ACCEPT

#  Allow HTTP and HTTPS connections from anywhere (the normal ports for websites and SSL).
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT

#  Allow SSH connections
#  The -dport number should be the same port number you set in sshd_config
-A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT

#  Allow ping
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

#  Log iptables denied calls
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

#  Reject all other inbound - default deny unless explicitly allowed policy
-A INPUT -j REJECT
-A FORWARD -j REJECT

COMMIT
```

iptables-persistentの設定は、サーバーの起動時に読み込まれます。再起動せず適用する場合は、手動で読み込ませる必要があります：


```bash
iptables-restore < /etc/iptables/rules.v4
```

