---
title: "[WIP]パーキングブレーキの状態をLEDで表示する"
---

:::message alert
このチャプターは執筆途中です。
:::

# 本チャプターの内容
最初の一歩として、LEDを使ったシンプルな出力から始めましょう。
パーキングブレーキの状態をLEDで表示します。
パーキングブレーキがONの時はLEDは点灯し、OFFの時は消灯するような動作です。

# 準備するもの
- Arduino
- LED
- 抵抗(330~1kΩ程度)
- ブレッドボード
- ジャンパーワイヤー

# 回路図
Arduinoと準備した電子部品を以下のように接続します。
![](/images/mobiflight-msfs2020-build-controller/parking-break-led/breadboard.png =400x)
回路図で表すと、このようになります。
![](/images/mobiflight-msfs2020-build-controller/parking-break-led/circuit.png =700x)


# MobiFlightの設定

# 実行
1. MSFS2020を起動しておきます。
1. MSFS2020が起動したら、MobiFlightのスタートボタンをクリックします。
1. MSFS2020で適当なフリーフライトを開始します。
1. 飛行機のパーキングブレーキを画面上でON/OFFして正常にLEDが点灯/消灯することを確認します。


# まとめ
これでON/OFFの二値状態をLEDで表示できました。
今回のパーキングブレーキだけではなく、ギアの状態やストロボ灯インジケーターなど様々な部分に応用可能です！
