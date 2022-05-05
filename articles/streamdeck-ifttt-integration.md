---
title: "Stream DeckでIFTTT連携をする"
emoji: "👆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["streamdeck", "ifttt"]
published: true
---
# はじめに
## Stream Deckとは
[Elgato Stream Deck](https://www.elgato.com/ja/stream-deck)はライブストリーミング配信等での使用を主目的としたマクロキーパッドです。大きな特徴として各ボタンは液晶モニタになっており、ボタンのアイコンを自由に変えることができるという特徴があります。
![](https://storage.googleapis.com/zenn-user-upload/0b4b0c927414a739e3257f14.png)

## やりたいこと
私の自宅ではSwitchBotを使用してスマートホーム化をしているため、Stream Deckをスマートホームの集中管理コンソールとして使うことができれば便利だと思い、やってみることにしました。

## どうやって実現する？
Elgato Stream DeckにはデフォルトでOBSやYoutubeに関する機能が提供されていますが、その他に「プラグイン」と呼ばれる、様々なサードパティ開発者によって提供される機能があります。その中にIFTTTのAppletを呼び出すプラグインがあるため、それを使用してStream Deckから家電操作を実現します。
![](https://storage.googleapis.com/zenn-user-upload/8b616d8c4bacc37b3a4dab85.png)

# やってみる
## 1.IFTTTの操作
:::message
IFTTTへのサインアップとSwitchBotとのアカウント連携は完了している前提です。
:::

IFTTTでAppletを作成して必要な情報２つを入手します。
- Event Name
- Maker Key

### 1-1.Appletの作成
トリガーで「Webhook」を選択してEvent Nameに名前を付けます。
**このEvent Nameは後ほど使用するのでメモしておきます。**
![](https://storage.googleapis.com/zenn-user-upload/a85873d9fbee66fe74cf76a3.png)

SwitchBotのアクションを追加・設定してAppletを作成します。
![](https://storage.googleapis.com/zenn-user-upload/77b00747a98a747caf3acee7.png)

### 1-2.WebhookからMaker keyの取得
作成したAppletのWebhookマークをクリックします。
![](https://storage.googleapis.com/zenn-user-upload/65ff8f0f3b870c5e4b2c0e30.png)

Webhookトップから「Settings」をクリックします。
![](https://storage.googleapis.com/zenn-user-upload/9304f72c25fd60baa9f358f0.png)

**「URL」の`https://maker.ifttt.com/use/`以下のランダム文字列がMaker Keyとなるのでメモします。**
![](https://storage.googleapis.com/zenn-user-upload/86dc44da4825f7c3419f1641.png)

## 2.Stream Deckの設定
### 2-1.IFTTTプラグインのインストール
Stream Deck Storeを開きます。
![](https://storage.googleapis.com/zenn-user-upload/a186f97366c9bd0760bec7c3.png)
プラグイン→IFTTTの「インストール」をクリックしてIFTTTプラグインをインストールします。
![](https://storage.googleapis.com/zenn-user-upload/4e8e9ece5dd8e5d8fc9c6ea0.png)

### 2-2.IFTTTボタンの追加＆設定
Stream Deckの空きスロットに「カスタム」カテゴリからIFTTTのプラグインをドラック＆ドロップします。
![](https://storage.googleapis.com/zenn-user-upload/8b80d405f71c603fa4cee4ec.png)

ドロップしたIFTTTプラグインの設定項目でIFTTTからコピーした**Maker Key**と**Event Name**を設定します。
![](https://storage.googleapis.com/zenn-user-upload/20c04445aba01a5bc573dea4.png)


## 完成！
好きなアイコンを設定して完成です。実際に押してみてテストします。
![](https://storage.googleapis.com/zenn-user-upload/9b86cd928147a6482c01f0d5.png)

## Tips
- Event Nameが同じWebhookを複数作成して呼び出すと複数のAppletを同時起動できます。
- （今後使用して気づいた点を追記していきます。）

# 最後に
これでStream Deckを使用してスマートホームを集中管理できるようになりました。
Stream Deckのフォルダ機能を使用して部屋ごとにフォルダを分けるなど様々な応用が考えられます。
IFTTTは無料ユーザーだとAppletの上限が３つまでなので、しばらく使ってみて便利そうなら有料プランを契約してすべてのSwitchBotと連携する予定です。