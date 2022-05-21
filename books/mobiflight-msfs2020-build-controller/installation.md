---
title: "MobiFlightのインストールと準備"
---

:::message
#### 前提条件
初めてArduinoを使用する場合はArduinoとPCをUSBで接続し、Windows用のドライバーを使用してボードを認識させる必要があります。
Windows 10の場合、ドライバーは既定でOSにインストールされてるため、ArduionoとPCをUSB接続するだけで認識します。
古いWindowsを使用する場合やドライバーを削除してしまった場合は[Arduinoソフトウエアのインストールプロセス](https://docs.arduino.cc/software/ide-v1/tutorials/Windows#toc4)を実行してドライバーをインストールします。

[参考：公式サイト](https://www.mobiflight.com/en/documentation/module.html#installation)
:::

# MobiFlightのインストール
1. [MobiFlightのダウンロードページ](https://www.mobiflight.com/en/download.html)からMobiFlightのインストールファイルをダウンロードします。
    ![](/images/mobiflight-msfs2020-build-controller/installation/download.png)
1. ダウンロードした `MobiFlight-Installer.exe` インストールファイルを実行してインストールします。インストーラーを起動するだけで特に設定項目などなくインストールが完了します。
    :::message alert
    インストーラーを起動すると、インストーラーファイルが置かれているフォルダにMobiFlight用のファイルが大量に展開されます。（インストール先は指定できません。）
    そのため、ダウンロードフォルダなど汎用フォルダ内で起動せず、MobiFlight用のフォルダを作成してそこで起動することをお勧めします。
    :::
    以下のようなWelcome画面が表示されればインストールは完了です。
    ![](/images/mobiflight-msfs2020-build-controller/installation/install-succeed.png)
1. 続けてMSFS2020用のWASMをインストールするか問われるのでインストールしておきます。
![](/images/mobiflight-msfs2020-build-controller/installation/install-wasm.png)


# Arduinoへのファームウェア書き込み
1. ArduinoをUSBケーブルで接続します。PCに直接接続するか、可能であれば外部電源を備えたUSBハブに接続してください。
1. Arduinoを初めてPCに接続する場合は、ボードが認識されるようにWindows用のドライバーをインストールする必要があります(前提条件に記載の内容を参照してください)。
1. インストールしたMobiFlight Connectorを起動します。
![](/images/mobiflight-msfs2020-build-controller/installation/launch-connector.png)
1. MobiFlight Connectorが起動するとMobiFlight用ボードとして使用可能な( = MobiFlightファームウェアがまだ書き込まれていない)Arduinoボードのスキャンが開始します。
![](/images/mobiflight-msfs2020-build-controller/installation/scan-arduino.png)
1. Arduinoが互換性のあるデバイスとして検出されます。ファームウェアをアップロードするか問われるので`OK`を選択してMobiFlightファームウェアをArduinoにアップロードします。
![](/images/mobiflight-msfs2020-build-controller/installation/upload-firmware.png)
1. 以下のような画面が表示されたらセットアップ完了です。
![](/images/mobiflight-msfs2020-build-controller/installation/compleate-prepare.png)

# まとめ
これで、MobiFlight用のArduinoボードができました!
いよいよ次のチャプターから実際にMSFS2020と連携をしてみます。

# コラム
#### Arduino用のボードに戻したい場合は？
MobiFlightファームウェアを書き込んだArduinoが不要になり、通常のArduinoとして使いたい場合はどうすればいいのでしょうか。
実はMobiFlightファームウェアは実態はただのArduinoのスケッチです。
つまり、ブートローダーを書き換えたりしているわけではないためMobiFlightファームウェアをアップロードしたArduinoへArduino IDEを使用してスケッチをアップロードすることにより、再び通常のArduinoボードとして使用できます。
[参考：公式フォーラム](https://www.mobiflight.com/forum/topic/217.html)