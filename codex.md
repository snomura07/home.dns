# Docker DNS (BIND) on AlmaLinux 9 — weather_report name resolution

## 目的
家庭LAN内のクライアント（PC1..n, 192.168.137.*/24）が、DNSサーバ（PC2: `192.168.137.11/24`）に問い合わせることで
ローカル名 `weather_report` を名前解決できるようにする。

- 目標: `weather_report` → `192.168.137.11` に解決できる
- アプリは `192.168.137.11:8088` で稼働している想定 
    つまり `weather_report` を引けるようにした上で、アクセスは `http://weather_report:8088/` のように行う。

## 前提 / 要件
- DNS端末（DNSサーバホスト）のIP: `192.168.137.11`（固定）
- Docker + Docker Compose で実装
- OSイメージ（ホスト想定）: AlmaLinux 9
- DNSサーバはLAN向けに `53/udp` と `53/tcp` を公開する
- 外部ドメイン（例: google.com）はフォワードで解決できるようにする（上流DNS: 8.8.8.8 / 8.8.4.4 など）
- README.md に構成・起動・確認手順を記載

---

## 推奨ディレクトリ構成（リポジトリ）
```text
home.md/
  docker-compose.yml
  named.conf
  zones/
    db.lan
  cache/          # bindのキャッシュ用
  logs/           # ログ用
　codex.md	　# このファイル
  README.md       # 実装内容をまとめるよう
