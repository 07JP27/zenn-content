---
title: "[WIP]無線周波数を7セグメントLEDで表示する"
---

:::message alert
このチャプターは執筆途中です。
:::

# 本チャプターの内容

# 準備するもの
- Arduino
- 7セグメントLED部品
- ジャンパーワイヤー
- ブレッドボード(必要に応じて)

# 7セグメントLED部品について
ディスプレイモジュールはMAX7219チップをベースにしている必要があります。MobiFlightコネクタソフトウェアで制御できるのはこのチップだけです。Amazonなどで「MAX7219」と検索すると、MAX7219チップと7セグメントLEDがバンドルされた以下のような部品(コンポーネント)を購入できます。
MAX7219は[SPI通信](https://emb.macnica.co.jp/articles/8191/)でArduinoと通信をします。MAX7219の実装について興味がある場合は[MAX7219のデータシート](http://www.microtechnica.tv/support/manual/MAX7219_jp.pdf)を確認してください。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/7seg-led.png)
[出典：Amazon](https://www.amazon.co.jp/s?k=Max7219&__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=3M9ZROTVXUXAF&sprefix=max7219%2Caps%2C235&ref=nb_sb_noss_1)


# 回路図
Arduinoと準備した電子部品を以下のように接続します。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/breadboard.png)
接続部を拡大するとこのような接続です。実際の部品にもDIN/CS/CLKなど同じようなラベルがプリントされていると思います。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/zoom.png)
回路図で表すと、このようになります。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/circuit.png)
※部品内部（MAX7219チップと7セグメントLED間）の接続は省略

# MobiFlightの設定

# 実行

# まとめ
