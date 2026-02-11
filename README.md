# tailscale-smb

ConoHa VPS のファイルシステムを macOS Finder から SMB でアクセスするための Docker Compose 構成。

Tailscale VPN トンネル経由でのみ接続可能（公開IPには露出しない）。

## アーキテクチャ

```
macOS Finder
    ↓ SMB (port 445)
[Tailscale VPN トンネル]
    ↓
[samba コンテナ] ←network_mode: service:tailscale→ [tailscale コンテナ]
    ↓ volume mount
/srv/renku, /srv/caddy-proxy, /home/deploy（ホスト）
```

## セットアップ

### 1. Tailscale Auth Key の取得

1. https://login.tailscale.com/admin/settings/keys にアクセス
2. 「Generate auth key」→ Reusable: ON / Expiration: 90日
3. 生成されたキーをメモ

### 2. デプロイ

```bash
cd /srv/tailscale-smb
cp .env.example .env
# .env を編集して TS_AUTHKEY と SAMBA_PASSWORD を設定
docker compose up -d
```

### 3. 起動確認

```bash
docker compose ps
docker exec ts-samba tailscale status
```

### 4. macOS Finder から接続

1. Finder → Cmd+K（サーバへ接続）
2. `smb://conoha-vps`（MagicDNS有効時）または `smb://<Tailscale IP>`
3. ユーザー名: `deploy` / パスワード: .env の `SAMBA_PASSWORD`

## 共有フォルダ

| 共有名      | ホストパス         |
| ----------- | ------------------ |
| renku       | /srv/renku         |
| caddy-proxy | /srv/caddy-proxy   |
| home        | /home/deploy       |

## セキュリティ

- Samba は Tailscale ネットワーク上でのみリスン（公開IP非露出）
- SMB3 のみ許可（SMB1/2 無効）
- ゲストアクセス無効
- `.env` は `.gitignore` で除外
