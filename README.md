# ARrow

駅の案内板を撮るだけで、今いる場所と向きが分かり、乗り換えの改札まで AR の矢印で案内する Android アプリです。DevFest GDGoC Japan Hackathon 2026（東京会場、チーム「ポーカー_AI木曜会」）の提出物です。

複雑な駅の構内で「今どこにいて、どちらへ進めばよいか分からない」を解消し、訪日観光客などの乗り換えを助けることを目指しています。いまは新宿駅の一部の経路で試作と検証を進めています。

## 主な機能

- 案内板の読み取り: カメラで撮った案内板の文字を Gemini が読み取り、サーバーが登録済みの案内板と照合する。縦向きでも横向きでも撮れる
- 現在地と向きの推定: 案内板の四隅の写り方と寸法から、撮った位置と向きを求め、経路をカメラの映像に重ねる
- AR の案内: 進む向きの矢印、次の曲がり角までの距離、残りの距離、到着を表示する
- 地下での位置の追跡: ARCore が位置を見失いやすい地下でも、ジャイロと歩数で位置を進める。道なりに歩いているときは、位置と向きを経路に沿わせてずれを抑える
- 外れたときの案内: 経路から外れたら戻る向きを指し、位置を見失ったら案内板の読み取り直しを促す
- Google マップの経路からの案内: Google マップで調べた経路をアプリに共有するか、経路のリンクを貼り付けると、乗り換える駅と路線を読み取る。地図の API は使わず、共有されたリンクだけを読む
- 表示の言語: 日本語・英語・中国語・韓国語

## 動かし方

### APK を入れる

1. [Releases](https://github.com/Devfest-Hackathon-2026/df-2026-t148/releases) から APK をダウンロードし、ARCore に対応した Android 端末（Android 10 以上）に入れる
2. アプリを開き、最初の画面で言語を選んで「はじめる」を押す
3. 駅の一覧から「新宿」を選び、乗り換える路線に「小田急線」を選んで「案内を始める」を押す
4. 新宿駅の地下1階、JR 9・10番線の階段の近くで、近くの案内板にカメラを向けて「読み取る」を押す
5. 現在地が分かると案内の画面に変わるので、矢印に沿って歩く。小田急線の乗り換え改札に着くと「着きました」と出る

案内できるのは、新宿駅の地下1階、JR 9・10番線の階段から小田急線の乗り換え改札までの経路（約 160m）です。案内板の読み取りには、インターネットへの接続が要ります。

### 自分でビルドする

- Unity 6000.3.28f1（Android Build Support を含む）でプロジェクトのフォルダを開く
- 次のコマンドで APK を作る。経路は環境変数 `EKINAVI_ROUTE`、サーバーの URL は `EKINAVI_SERVER_URL` で指定する

```
EKINAVI_ROUTE=b1_9_10_to_odakyu EKINAVI_SERVER_URL=<サーバーの URL> \
Unity -batchmode -quit -buildTarget Android -projectPath . \
  -executeMethod EkiNavi.EditorTools.SignDemoBatchBuilder.BuildApk -logFile -
```

サーバーを手元で動かす方法と API は [server/README.md](server/README.md)、Cloud Run に出す手順は [docs/cloud-run-deploy.md](docs/cloud-run-deploy.md) にあります。

## 構成

| 部分 | 技術 | 役割 |
|---|---|---|
| アプリ | Unity 6、AR Foundation、ARCore（C#） | カメラで案内板を撮る、現在地と向きの推定、推測航法、AR の矢印 |
| サーバー | Python、FastAPI、NetworkX | 案内板の識別（`POST /v1/sign/identify`）、案内板の一覧（`GET /v1/station`）、経路パック |
| AI | Gemini（Vertex AI 経由） | 案内板の文字・外枠・四隅の読み取り |
| インフラ | Cloud Run、Cloud Build、Artifact Registry、Cloud IAM | サーバーの配置。API キーはアプリに持たせず、サーバーのサービスアカウントで Gemini を呼ぶ |

## フォルダ

| フォルダ | 中身 |
|---|---|
| `Assets/EkiNavi` | アプリのコード、シーン、経路、画面 |
| `server` | サーバー（`app`）、経路パックの生成（`pack`）、テスト |
| `contracts` | アプリとサーバーの間の取り決め（API、案内板の形式） |
| `docs` | Cloud Run の手順、画面の見本 |

## データと出典

- 経路は、国土交通省「新宿駅周辺屋内地図オープンデータ」（[G空間情報センター](https://www.geospatial.jp/ckan/dataset/mlit-indoor-shinjuku-r2)、政府標準利用規約）を加工して作成しています
- 案内板の位置や文字などのデータは、サーバーにだけ置き、このリポジトリには入れていません。アプリは起動時にサーバーから案内板の一覧を読みます
- 駅の写真は Wikimedia Commons のものです。撮影者とライセンスはアプリの「写真と地図の出典」に載せています
