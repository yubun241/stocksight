# StockSight PWA — デプロイ手順 (GitHub Pages → Android)

藤井工藝 / Fujii Kogei

## ファイル構成
```
index.html       アプリ本体 (HTML/CSS/JS 一体)
manifest.json    PWAマニフェスト
sw.js            Service Worker (ネットワーク優先)
icon-192.png     アイコン
icon-512.png     アイコン
```

## 1. GitHub Pages へ公開
GT DASH と同じ手順です。

```bash
# 例: yubun241.github.io リポジトリ内に stocksight/ フォルダを作る場合
cd yubun241.github.io
mkdir stocksight
cp index.html manifest.json sw.js icon-*.png stocksight/
git add . && git commit -m "Add StockSight PWA" && git push
```

→ `https://yubun241.github.io/stocksight/` で公開されます。

※ サブフォルダ運用のため manifest の start_url / scope は相対パス (./) 済み。

## 2. Android で起動
1. Chrome で上記URLを開く
2. メニュー →「ホーム画面に追加」(または自動表示されるインストールバナー)
3. ホームのアイコンから全画面アプリとして起動

## 3. Google Play 配信 (TWA化) — 任意
GT DASH と同じ bubblewrap フローが使えます。

```bash
npm i -g @bubblewrap/cli
bubblewrap init --manifest https://yubun241.github.io/stocksight/manifest.json
bubblewrap build
# 署名は既存キーストア (~/android.keystore, alias: android) を指定
```

assetlinks.json をルートドメイン (`yubun241.github.io/.well-known/`) に追記して
URLバーを消すのも GT DASH と同様です。

## CORSプロキシについて
ブラウザから Yahoo Finance を直接呼ぶと CORS で拒否されるため、
無料の中継プロキシ (corsproxy.io → allorigins.win の順に自動フォールバック) を経由します。

安定運用したい場合は、お手元の Vercel (stock-analyzer-api) に
以下のような中継エンドポイントを1本生やし、アプリの 設定 → CORSプロキシ → カスタム に
`https://<your-app>.vercel.app/api/proxy?url=` を指定してください。

```python
# api/proxy.py (Vercel Python Functions)
from http.server import BaseHTTPRequestHandler
from urllib.request import Request, urlopen
from urllib.parse import urlparse, parse_qs

class handler(BaseHTTPRequestHandler):
    def do_GET(self):
        url = parse_qs(urlparse(self.path).query).get('url', [''])[0]
        if not url.startswith('https://query1.finance.yahoo.com/'):
            self.send_response(403); self.end_headers(); return
        req = Request(url, headers={'User-Agent': 'Mozilla/5.0'})
        body = urlopen(req, timeout=15).read()
        self.send_response(200)
        self.send_header('Access-Control-Allow-Origin', '*')
        self.send_header('Content-Type', 'application/json')
        self.end_headers()
        self.wfile.write(body)
```

## 課金について
外部有料サービス・APIキーは一切使用していません。
