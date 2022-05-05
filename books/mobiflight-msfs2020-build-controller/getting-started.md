---
title: "はじめに"
---
# まえがき
狭い空間にところ狭しと並べられたトグルスイッチやノブ、夜にはオレンジ色のバックライトに照らされる飛行機のコックピットはガジェット好きであれば誰しも一度は憧れたことがあるのではないでしょうか。
![](/images/mobiflight-msfs2020-build-controller/getting-started/cockpit.png)

この世にはフライトシミュレーターと呼ばれるゲームが存在し、忠実に再現された飛行機をパソコンの中で操縦することができ、特に代表的なフライトシミュレーターとして Microsoft Flight Simulator 2020 があります。MSFS2020とも略されるこのゲームはダイナミックで生き生きとした世界での飛行、リアルタイムの大気シミュレーションなど、飛行機の機体のみならず周囲の環境も含めてさまざまなシチュエーションを再現できます。(実は上の画像もMSFS2020のキャプチャ画像です)
https://www.xbox.com/ja-JP/games/microsoft-flight-simulator


ただ、非常にリアルなMSFS2020にも足りないものがあります。
そう、パチパチとON/OFFができるトグルスイッチやカリカリと回すノブの感覚はMSFS2020だけでは再現できません。MSFS2020を更にリアルなものにするために物理操作デバイス（コントローラー）を作成するのが本書の目的です。


さぁ、MSFS2020をさらにリアルするための１歩を踏み出しましょう！

# 本書に記載している内容
MobiFlightとArduinoを使用してMicrosoft Flight Simulator 2020の物理操作デバイス（コントローラー）を作成する方法を解説します。
前半は各入出力の要素を使用する方法を一要素づつ解説し、後半ではそれらを合わせて実際にフライトに使用するコントローラーを作成します。
なお、本書では筐体の制作に3Dプリンターを使用していますが、3Dプリンターは必ずしも必要ではありません。代わりにMDF板を使用するなど、他の方法でも創意工夫によって可能性は無限大です！

# 本書に記載していない内容
本書ではArduinoとはなにか？や電子部品の動作原理（例えばロータリーエンコーダーはどのように動作するか）などの電子部品の基礎知識については説明していません。

# MobiFlightとは？
MobiFlightは、標準的なハードウェアをフライトシミュレータに統合するソフトウエアで、以下のような特徴があります。
#### 簡単
MobiFlight Connectorを使用すると、スクリプトやコードを1行も使用せずに、マウスをクリックするだけでMobiFlightモジュールを簡単に設定できます。
#### リーズナブル
MobiFlightは一般的に入手可能なハードウェアで動作するため、コストを大幅に削減することができます。MobiFlightは、LED、7セグメント、ステッパー、サーボ、LCDなどをサポートしています。
#### フレキシブル
本書ではMSFS2020を使用しますが、以下のフライトシミュレーターでも使用することができます。
- MSFS2020
- P3D
- FSX
- FS9
- X-Plane11
- X-Plane10
- X-Plane9

そしてフライトシミュレーターだけでなく、Arduinoシリーズや電子部品を自由に選択して自分だけのコントローラーを作成することができます。
#### オープンソース
MobiFlightは100％オープンソースであり、コミュニティによって維持されている他のプロジェクトとともに[GitHubでホスト](https://github.com/MobiFlight)されています。


https://www.mobiflight.com/en/index.html


# 前提条件
## MobiFlight
本書内で使用するMobiFlightは執筆時点で最新バージョンの9.3.2を使用しています。

## Arduino
MobiFlightはArduinoを使用して電子部品とMSFS2020の連携を行いますが、すべてのArduinoシリーズが使用できるわけではありません。本書ではArduino Unoを使用していますが、安定しない場合はArduino Megaの使用を検討してください。
#### サポートされているArduinoシリーズ
- [Arduino Mega 2560 R3](https://docs.arduino.cc/hardware/mega-2560) もしくは互換ボード
- [Arduino Mega Pro Mini](https://www.amazon.co.jp/WINGONEER-Arduino-MINI%E3%82%A8%E3%83%B3%E3%83%99%E3%83%87%E3%83%83%E3%83%89%E3%80%81MCU-ATmega2560%E3%80%81USB-CH340G%E3%82%A8%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AD%E3%83%8B%E3%82%AF%E3%82%B9/dp/B07HBR257M) もしくは互換ボード
- [Sparkfun Micro Pro](https://www.switch-science.com/catalog/1623/) もしくは互換ボード

#### 実験的にサポートされているArduinoシリーズ
- [Arduino Uno](https://docs.arduino.cc/hardware/uno-rev3)：現時点では安定していません。もし問題が発生した場合は、代わりにArduino Megaを入手してください。

#### サポートされて`いない`Arduinoシリーズ
- Arduino Mini
- Arduino Micro
- Arduino Pro Mini
- Arduino Nano
- Arduino Leonardo
- その他リストに無いArduinoシリーズ

[参考:公式サイト](https://www.mobiflight.com/en/documentation/module.html)

## その他
- 本書はPCにMSFS2020がインストールされている前提で記載されています。未インストールの場合はインストール後に本書の内容を試してください。