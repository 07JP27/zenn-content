---
title: "Stream DeckでSwitchBotを操作する(SwitchBotAPI版)"
emoji: "🔘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["streamdeck", "switchbot"]
published: true
---
:::message
この記事に記載されている方法はWindowsに接続されたStream Deckでのみ動作します。(が、エッセンスはMacでも使用できると思います。)
:::
# はじめに
先日以下の記事でIFTTTと連携してStream DeckからSwitchBotを操作する方法について書きました。
https://zenn.dev/07jp27/articles/streamdeck-ifttt-integration

その記事の中で
> IFTTTは無料ユーザーだとAppletの上限が３つまでなので、しばらく使ってみて便利そうなら有料プランを契約してすべてのSwitchBotと連携したいと思います。

と書いたものの、`SwitchBotのAPIがあれば直接呼べるのでは？`と考えていました。
実は2020年6月に初めてSwitchBotを購入した際にAPIの提供について調べたことがあり、そのときはAPIが提供されていなかったので期待はしてませんでした。
しかし調べてみたところ、なんと2020年の12月にAPIのドキュメントページが公開されていました！🎉

https://github.com/OpenWonderLabs/SwitchBotAPI

というわけで今回はIFTTTを使用せずにStream Deckから直接SwitchBotAPIをCallしてSwitchBotに接続された機器を操作してみます。

# やってみる

## SwitchBotAPIトークンの取得
SwichBotAPIのトークンはSwichBotアプリから取得することができます。
1. SwichBotアプリの「設定」を開いて **「アプリバージョン」を10回タップ** します。
![](https://storage.googleapis.com/zenn-user-upload/9a33e5da76de737d19d105df.png =300x)

1. １０回タップすると表示される「開発者向けオプション」というメニューをタップします。
![](https://storage.googleapis.com/zenn-user-upload/446394072707a57545da63da.png =300x)

1. トークンが表示されるのでメモしておきます。
![](https://storage.googleapis.com/zenn-user-upload/d31421a4b4d7d39ac830cada.png =300x)

## 操作する機器のデバイスIDの取得
[Postman](https://www.postman.com/product/api-client/)や[Insomnia](https://insomnia.rest/products/insomnia)でURLを`https://api.switch-bot.com/v1.0/devices`、Headerに`Authorization：SwitchBotアプリで取得したトークン`をセットしてAPIを呼び出します。(CLI使いたい人はもちろんcurlなどでもOKです。)
リクエストに成功するとトークンのアカウントに紐付いている機器一覧が取得できるので操作したい機器の`deviceId`をメモしておきます。

![](https://storage.googleapis.com/zenn-user-upload/b599aab6fe5cb9fb7affa7df.png)

## Stream Deckの設定
### API Ninjaプラグインのインストール
1. Stream Deck Storeを開きます。
![](https://storage.googleapis.com/zenn-user-upload/6b99e8d00cb435a26bebbe2f.png)
1. プラグイン→開発ツール→API Njnjaの「インストール」をクリックします。
![](https://storage.googleapis.com/zenn-user-upload/cb067d1457b193dd267232ec.png)

### API Ninjaの設定
1. BarRaiderカテゴリからAPI Ninjaをドラック＆ドロップします。
![](https://storage.googleapis.com/zenn-user-upload/62d0d66b7ced1f2a647c0a51.png)

1. [SwitchBotAPIのドキュメント](https://github.com/OpenWonderLabs/SwitchBotAPI)の通りAPIの呼び出し設定を行います。
**ここではサンプルとしてPlugの電源オンを行います。**

| 項目 |設定 |
| ---- | ---- |
| Request Type  | POST |
| API URL | `https://api.switch-bot.com/v1.0/devices/{取得したPlugのデバイスID}/commands` |
| Content Type  | application/json |
| Headers  | `Authorization:{SwitchBotアプリから取得したトークン}` |
| Data  | `{"command": "turnOff","parameter": "default","commandType": "command"}` |




![](https://storage.googleapis.com/zenn-user-upload/f54ea36ddcf42a1155cbae29.png)
![](https://storage.googleapis.com/zenn-user-upload/d9833f795ece8beb5990c352.png)

## 完成！
好きなアイコンを設定して完成です。実際に押してみてテストします。

# 最後に
これでIFTTTを使用せずにSwitchBotの機器を操作することができ、有料プランからの脱出に成功しました。
今回使用したプラグインのAPI NinjaがWindowsのみでしかリリースされていなかったため、本記事の方法ではWinsowsでしか設定することはできませんが、内部処理としてはURLにアクセスしているだけなのでMacでも実現できそうです。
SwitchBot公式でStream Deckプラグインがリリースされるのを(あまり期待せずに)待ちつつ、自分でもプラグインを作ってみようと思いました（まだSDKも無いようなのでそれも含めて実装してみようかと思います）。