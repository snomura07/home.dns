# Docker DNS + Nginx Reverse Proxy on AlmaLinux 9

家庭 LAN 内の入口を 1 台 (`192.168.137.11`) に集約し、
ホスト名ごとに各アプリへリバースプロキシする構成です。

- DNS サーバホスト: `192.168.137.11`
- DNS: BIND `9.20` 系（`9.18.43` 以前の脆弱性回避のため）
- Reverse Proxy: Nginx (`80/tcp`)
- DNS 方針: `*.home -> 192.168.137.11`（入口固定）

## 運用方針（固定）

- クライアントは常に `*.home` で `192.168.137.11` に到達する
- 実際のアプリ配置先（同一PC / 別PC）は Nginx の `proxy_pass` で切り替える
- 新規アプリ追加時、DNS への個別追記は基本不要

例:
- `todo.home` -> `192.168.137.11`（DNS）-> `192.168.137.90:9000`（Nginx転送）

## ディレクトリ構成

- `docker-compose.yml`: DNS + Nginx 定義
- `named.conf`: BIND メイン設定
- `zones/db.home`: `home` ゾーン（`* IN A 192.168.137.11`）
- `nginx/nginx.conf`: Nginx メイン設定
- `nginx/conf.d/*.conf`: アプリごとの vhost 設定

## 起動

```bash
docker compose up -d
```

## 初期確認

1. DNS 確認

```bash
nslookup weather_report.home 192.168.137.11
nslookup google.com 192.168.137.11
```

2. HTTP 確認

```bash
curl -I http://weather_report.home/
```

## 新しいアプリケーションの追加方法（推奨手順）

以下は `todo.home` を別PC `192.168.137.90:9000` に転送する例です。

1. `nginx/conf.d/todo.conf` を追加

```nginx
server {
  listen 80;
  server_name todo.home;

  location / {
    proxy_pass http://192.168.137.90:9000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

2. Nginx 設定テスト

```bash
docker compose exec reverse-proxy nginx -t
```

3. 反映

```bash
docker compose exec reverse-proxy nginx -s reload
```

4. 動作確認

```bash
nslookup todo.home 192.168.137.11
curl -I http://todo.home/
```

## 同一PC上アプリへの転送例

DNS/Nginxホスト自身にあるアプリ (`8088` など) へは次のように設定します。

```nginx
proxy_pass http://host.docker.internal:8088;
```

## 注意点

- `*.home` はすべて `192.168.137.11` に解決されます（入口固定）
- 別PCアプリへ転送する場合は、転送先PCのFWで該当ポートを許可してください
- 同名の明示Aレコードがある場合、ワイルドカードより明示レコードが優先されます
