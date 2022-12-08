---
title: "Azureで作る「おうちデジタルツイン」"
emoji: "🏡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "iot", "digitaltwin", "switchbot"]
published: false
---

この記事は Azure Advent Calendar 2022 の 25日目の記事です。
https://qiita.com/advent-calendar/2022/azure

# はじめに
Azureでデジタルツインソリューションを構築する場合はAzure Digital Twinsがあるが、それだけではなくUI層までAzureを使って行く。
## 作ったもの

## この記事は何であって何ではないか
である
- Azureの各種サービスを使用して個人レベルでもエンドツーエンドのデジタルツインソリューションが構築できます！という紹介

ではない
- 上記を構築するための手順やガイド（もしかしたら今後別途書くかも）

# アーキテクチャ
全体的なデータや操作の流れはこのような形です。（細かくてすみません。拡大してごらんください・・・）
画像には主にセンサーデータのトランザクショナルなデータの流れと３Dモデルのインポートなどスタティックなデータの流れが記載されています。
![](/images/home-digitaltwin-on-azure/architecture.png)
# 各要素のポイント
## エッジデバイス/ゲートウェイ
家庭用IoTデバイスの雄、SwitchBot

## Azure Digital Twins
### ツイングラフ
### データイングレス

### 3Dモデリング

## クライアントアプリケーション
### アプリの開発
3D Scene studio（まだプレビュー）
埋め込み
### 認証設定
Easy Auth


# 今回は実装しなかったが、今後使えそうな機能
## イベントルートの利用
イベントルートを使用するとツインの更新を別のツインに対して通知することができます。
例えば温湿度計ツインのプロパティが更新された際に、その温湿度計が存在している部屋ツインに対して温湿度を複製し、部屋全体の温湿度として扱うことができるようになります。
これにより現実世界をより正確に反映したモデルが実現できます。
![](/images/home-digitaltwin-on-azure/event-route.png)

具体的な仕組みとしては以下の通りEvent HubとAzure Functionsの組み合わせで実現できます。
![](/images/home-digitaltwin-on-azure/routing-workflow.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/digital-twins/concepts-route-events)
https://learn.microsoft.com/ja-jp/azure/digital-twins/concepts-route-events


## Digital Twins データの履歴化
Azure Digital Twins(のツイングラフ)は最新のプロパティを保持しますが、これを履歴化して時系列グラフで表示したい、という要望もあると思います。そこで利用できるのがAzure Data Explorer統合を利用したデータの履歴化です。これにより、Azure Digital TwinsインスタンスをAzure Data Explorerクラスターに接続し、デジタルツインのプロパティ更新をAzure Data Explorerに自動的に履歴化できます。
![](/images/home-digitaltwin-on-azure/data-history-architecture.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/digital-twins/concepts-data-history)
https://learn.microsoft.com/ja-jp/azure/digital-twins/concepts-data-history

この方法を使用すると時系列データがAzure Data Explorerに保存されるので、そこからKustoクエリを使用して時系列データを取得することができます。下の画像はAzure Portal上でクエリをかけている様子ですが、[Azure Data Explorer REST API](https://learn.microsoft.com/ja-jp/azure/data-explorer/kusto/api/rest/)を使用して自分のアプリでデータを使用することも可能です。
![](/images/home-digitaltwin-on-azure/data-history-run-query-2.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/digital-twins/how-to-use-data-history)

https://learn.microsoft.com/ja-jp/azure/digital-twins/how-to-use-data-history

## SwitchBot Webhook の利用
SwitchBotにはAPI V1.1からWebhookが追加されました。これを使えばリアクティブにデバイスの状態変化を検知できます。
しかし、今回は対応していないデバイスが多かったこと、セキュリティ的に不安があったことからWebhookではなくポーリング方式としました。Webhookが整備されてくればそちらを使いたいと思います。
なぜWebhookの採用に至らなかったのはか以下のスクラップで記載しています。
https://zenn.dev/07jp27/scraps/4f2ff5aceff79d

# 今後の展望（拡張）
## 家１軒まるごとデジタルツイン化
## 操作コマンド
## 他のメーカーのIoTデバイス

# 参考
今回のおうちデジタルツインを作るにあたって調査したAzure Digital Twinsやその周辺情報は以下のスクラップにまとめています。
https://zenn.dev/07jp27/scraps/d7548193837c70
