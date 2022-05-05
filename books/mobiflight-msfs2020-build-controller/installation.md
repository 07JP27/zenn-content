---
title: "MobiFlightのインストールと準備"
---

:::message
#### 前提条件
初めてArduinoを使用する場合はArduinoとPCをUSBで接続し、Windows用のドライバーを使用してボードを認識させる必要があります。Windows 10の場合、ドライバーは既定でOSにインストールされてるため、ArduionoとPCをUSB接続するだけで認識します。
古いWindowsを使用する場合やドライバーを削除してしまった場合は[Arduinoソフトウエアのインストールプロセス](https://docs.arduino.cc/software/ide-v1/tutorials/Windows#toc4)を実行してドライバーをインストールします。

[参考：公式サイト](https://www.mobiflight.com/en/documentation/module.html#installation)
:::

# MobiFlightのインストール
1. [MobiFlightのダウンロードページ](https://www.mobiflight.com/en/download.html)からMobiFlightのインストールファイルをダウンロードします。
1. インストールファイルを実行してウィザードに従ってインストールします。


# Arduinoへのファームウェア書き込み
1. ArduinoをUSBケーブルで接続します。PCに直接接続するか、可能であれば外部電源を備えたUSBハブに接続してください。
1. Arduinoを初めてPCに接続する場合は、ボードが認識されるようにWindows用のドライバーをインストールする必要があります(前提条件に記載の内容を参照してください)。
1. インストールしたMobiFlight Connectorを起動します。
1. Arduinoが互換性のあるデバイスとして検出されます。
1. Mobiflight Connectorの組み込みアップロード機能を使用してMobiFlightファームウェアをArduinoにアップロードします。

これで、MobiFlightボードができました!
いよいよ次のセクションから実際にMSFS2020と連携をしてみます。