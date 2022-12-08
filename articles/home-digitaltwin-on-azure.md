---
title: "Azureで作る「おうちデジタルツイン」"
emoji: "🏡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "iot", "digitaltwin", "switchbot"]
published: false
---

この記事は Azure Advent Calendar 2022 の 25日目の記事です。
https://qiita.com/advent-calendar/2022/azure

## はじめに
Azureでデジタルツインソリューションを構築する場合はAzure Digital Twinsがあるが、それだけではなくUI層までAzureを使って行く。

## この記事は何であって何でないか
である
- Azureの各種サービスを使用して個人レベルでも可能なエンドツーエンドのデジタルツインソリューション構築の事例紹介

でない
- 上記を構築するための手順やガイド（もしかしたら今後別途書くかも）

## アーキテクチャ

## 各要素のポイント
### エッジデバイス
### Azure Digital Twins
### クライアントアプリケーション

## 拡張性

## 今回やらなかったこと
### イベントルートの利用
### Digital Twins データの履歴化
Azure Digital Twins(のツイングラフ)は最新のプロパティを保持しますが、これを履歴化して時系列グラフで表示したい、という要望もあると思います。そこで利用できるのがAzure Data Explorer統合を利用した方法です。これにより、Azure Digital Twins インスタンスを Azure Data Explorer クラスターに接続し、デジタル ツイン プロパティの更新を Azure Data Explorer に自動的に履歴化できます。
![](/images/home-digitaltwin-on-azure/data-history-architecture.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/digital-twins/concepts-data-history)
https://learn.microsoft.com/ja-jp/azure/digital-twins/concepts-data-history

今回は構成していませんが、この方法を使用するとデータがAzure Data Explorerに保存されるので、そこからKustoクエリを使用して時系列データを取得することができます。
![](/images/home-digitaltwin-on-azure/data-history-run-query-2.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/digital-twins/how-to-use-data-history)

https://learn.microsoft.com/ja-jp/azure/digital-twins/how-to-use-data-history

### SwitchBot Webhook の利用
SwitchBotにはAPI V1.1からWebhookが追加されました。
しかし、今回は対応していないデバイスが多かったこと、セキュリティ的に不安があったことからWebhookではなくポーリング方式としました。Webhookが整備されてくればそちらを使いたいと思います。
なぜWebhookの採用に至らなかったのはか以下のスクラップで記載しています。
https://zenn.dev/07jp27/scraps/4f2ff5aceff79d

## 参考
今回のおうちデジタルツインを作るにあたって調査した情報は以下のスクラップにまとめています。
https://zenn.dev/07jp27/scraps/d7548193837c70