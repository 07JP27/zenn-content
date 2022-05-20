---
title: "はじめに"
---
# まえがき
狭い空間にところ狭しと並べられたトグルスイッチやノブ、夜にはオレンジ色のバックライトに照らされる航空機のコックピットはガジェット好きであれば誰しも一度は憧れたことがあるのではないでしょうか。
![](/images/mobiflight-msfs2020-build-controller/getting-started/cockpit.png)

この世にはフライトシミュレーターと呼ばれるゲームが存在し、忠実に再現された航空機をゲーム機やパソコンの中で操縦できます。
特にそのリアルさからフライトシミュレーターの代表格として取り上げられるのがMicrosoft Flight Simulator 2020です。MSFS2020とも略されるこのゲームはダイナミックで生き生きとした世界での飛行、リアルタイムの大気シミュレーションなど、航空機の機体のみならず周囲の環境も含めてさまざまなシチュエーションを再現できます。(実は上の画像もMSFS2020のキャプチャ画像です)
https://www.xbox.com/ja-JP/games/microsoft-flight-simulator


ただ、非常にリアルなMSFS2020にも足りないものがあります。
そう、パチパチとON/OFFができるトグルスイッチやカリカリと回すノブの感覚はMSFS2020だけでは再現できません。MSFS2020を更にリアルなものにするため、物理操作デバイス（コントローラー）を作成するのが本書の目的です。


さぁ、MSFS2020をさらにリアルするための１歩を踏み出しましょう！

# 本書の対象読者
すでにMSFS2020を使っている、もしくは使い始めた人で自作コントローラーの作成に興味がある人。
まだMSFS2020を使っていない人でもわかるように努めていますが航空機自体の操作方法を知ることにより、よりイメージしやすくなっています。

# 本書に記載している内容
MobiFlightとArduinoを使用してMicrosoft Flight Simulator 2020の物理操作デバイス（コントローラー）を作成する方法を解説します。
前半は各入出力の要素を使用する方法を一要素づつ解説し、後半ではそれらを合わせて実際にフライトに使用するコントローラーを作成します。
なお、本書では筐体の制作に3Dプリンターを使用していますが、必ずしも3Dプリンターが必要なわけではありません。代わりにMDF板を使用するなど、他の方法でも創意工夫によって可能性は無限大です！

# 本書に記載していない内容
- Arduinoとはなにか？の基本や電子部品の動作原理（例えばロータリーエンコーダーはどのように動作するか）などの電子工作の基礎知識について。
- MSFS2020でA320のオートパイロットを設定するにはどうすればよいか？などMSFS2020自体や航空機の操作方法

本書はArduinoを知らなくても良いように記述を心がけていますが、基礎を知っているに越したことはありません。Arduinoの基礎は[ドットインストール](https://dotinstall.com/lessons/basic_arduino)などのサイトで学習できます。

# MobiFlightとは？
MobiFlightは、標準的な電子部品をフライトシミュレータに統合するソフトウエアで、以下のような特徴があります。
#### 簡単
MobiFlight Connectorを使用すると、スクリプトやコードを1行も使用せずに、マウスをクリックするだけでMobiFlightモジュールを簡単に設定できます。
#### リーズナブル
MobiFlightは一般的に入手可能な電子部品で動作するため、コストを大幅に削減できます。MobiFlightは、LED、7セグメントLED、ステッピングモーター、サーボモーター、LCDなどをサポートしています。
#### フレキシブル
本書ではMSFS2020を使用しますが、以下のフライトシミュレーターでも使用できます。
- MSFS2020
- P3D
- FSX
- FS9
- X-Plane11
- X-Plane10
- X-Plane9

もちろんフライトシミュレーターだけでなく、Arduinoシリーズや電子部品を自由に選択して自分だけのコックピットを作成できます。
#### オープンソース
MobiFlightは100％オープンソースであり、コミュニティによって維持されている他のプロジェクトとともに[GitHubでホスト](https://github.com/MobiFlight)されています。

https://www.mobiflight.com/en/index.html

#### 公式フォーラム
公式フォーラムで高度な使用方法やQ&A、他のユーザーの事例を閲覧できます。もちろん自らスレッドを立てて発信も可能です。
https://www.mobiflight.com/forum.html


# 前提条件
## MobiFlight
本書内で使用するMobiFlightは執筆時点で最新バージョンの9.3.2を使用しています。

## Arduino
MobiFlightはArduinoを使用して電子部品とMSFS2020を連携しますが、すべてのArduinoシリーズが使用できるわけではありません。本書ではArduino Unoを使用していますが、安定しない場合はArduino Megaの使用を検討してください。
[参考:公式サイト](https://www.mobiflight.com/en/documentation/module.html)

#### サポートされているArduinoシリーズ
- [Arduino Mega 2560 R3](https://docs.arduino.cc/hardware/mega-2560) もしくは互換ボード
- [Arduino Mega Pro Mini](https://www.amazon.co.jp/WINGONEER-Arduino-MINI%E3%82%A8%E3%83%B3%E3%83%99%E3%83%87%E3%83%83%E3%83%89%E3%80%81MCU-ATmega2560%E3%80%81USB-CH340G%E3%82%A8%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AD%E3%83%8B%E3%82%AF%E3%82%B9/dp/B07HBR257M) もしくは互換ボード
- [Sparkfun Micro Pro](https://www.switch-science.com/catalog/1623/) もしくは互換ボード

#### 実験的にサポートされているArduinoシリーズ
- [Arduino Uno](https://docs.arduino.cc/hardware/uno-rev3)：現時点では安定していません。もし問題が発生した場合は、代わりに動作がサポートされているArduino Megaを使用してください。

#### サポートされて`いない`Arduinoシリーズ
- Arduino Mini
- Arduino Micro
- Arduino Pro Mini
- Arduino Nano
- Arduino Leonardo
- その他リストに無いArduinoシリーズ

## その他
- 本書はPCにMSFS2020がインストールされている前提で記載されています。未インストールの場合はインストール後に本書の内容を試してください。
- MSFS2020はWindowsのみサポートしているため、本書に記載の内容もWindowsでのみ動作します。(本書ではWindows 10を使用しています。)
