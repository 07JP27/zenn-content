---
title: "無線周波数を7セグメントLEDで表示する"
---
# 本チャプターの内容
ここまでの内容でON/OFFの出力と入力ができるようになりました。
より複雑な出力として無線周波数やオートパイロットパネルで高度や針路の表示に使われる7セグメントLEDを使用した出力をしてみます。
### 完成イメージ
スタンバイ無線周波数の表示を7セグメントLEDで表示します。
https://www.youtube.com/watch?v=uUOZhN9IrxE


# 準備するもの
- [Arduino](https://akizukidenshi.com/catalog/c/carduino1/)
- [7セグメントLED部品](https://www.amazon.co.jp/gp/product/B088CR8LR6)
- [ブレッドボード](https://akizukidenshi.com/catalog/g/gP-05294/)
- [ジャンパーワイヤー](https://akizukidenshi.com/catalog/g/gC-05159/)

# 7セグメントLED部品について
MobiFlightで7セグメントLEDを使用する場合は**MAX7219チップをベースにしている部品である必要があります**。MobiFlight Connectorで制御できるのはこのチップだけです。Amazonなどで「MAX7219」と検索すると、MAX7219チップと7セグメントLEDがバンドルされた以下のような部品(コンポーネント)を購入できます。
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
※部品内部（MAX7219チップと7セグメントLED間）の接続は省略しています。

# MobiFlightの設定
### デバイスの設定
1. MobiFlight Connectorを起動してメニューバーから `Extras`→`Settings`をクリックして`Settings`画面を開きます。`MobiFlight Modules`タブを開くと現在接続されているMobiFlightボードがリスト表示されます。
1. デバイスを追加したいMobiFlightボードを選択して`Add device`→`LED 7-Segment`をクリックします。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/1.png)
1. ボードに選択したデバイスが追加されます。接続の通り`DIN`を`7`、`CS`を`6`、`CLK`を`5`にセットし、アップロードボタンをクリックしてMobiFlightボードに変更を書き込みます。OKをクリックして`Settings`画面を閉じます。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/2.png)

### フライトシミュレーターとデバイスのマッピング
1. 新規Outputを作成して`COM`と名前をつけてedit列の`...`をクリックします。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/101.png)
1. ConfigWizardが表示されます。`Sim Variable`タブで`Select Preset`を`COM_RADIO_STBY_FREQUENCY` を選択します。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/102.png)
1.  ConfigWizardの`Display`タブに移動して以下の設定をします。
`Test`をクリックして設定通りの表示されるか確認します。
正常に点灯したら`OK`をクリックしてConfigWizardを閉じます。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/103.png)
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/104.png)

# 実行
1. MSFS2020を起動します。
1. MSFS2020が起動していればMobiFlight Connectorの`Run`ボタンがクリックできるようになるので、COMのマッピングがActiveになってることを確認して`Run`ボタンをクリックします。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/201.png)
1. MSFS2020で適当なフリーフライトを開始します。
1. 無線パネルで周波数変更ダイヤルを回し、7セグメントLEDの表示が連動することを確認します。

https://www.youtube.com/watch?v=uUOZhN9IrxE

# まとめ
7セグメントLEDを出力することにより、一気にコックピット感が増しました。
次のチャプターではこの表示をダイヤルで入力してみます。


# コラム
### デイジーチェーン
7セグメントLEDはホームコックピットには欠かせないデバイスですが、電源の２本以外にSPI通信のための3本の信号線が必要です。
たとえば無線周波数のパネルはアクティブとスタンバイ２つの7セグメントLEDが必要ですが、その場合、3＊2=6本のArduinoピンを使用することになり、特にデバイスが多いオートパイロットのパネルなどは、ピンに空きがない！という状況も多く発生することがあります。
それを解決するのが**デイジーチェーン**です。MobiflightでサポートされているMAX7219チップは最大８個までの7セグメントLEDをデイジーチェーンで連結してたった３本の信号線で最大8桁＊8デバイス=65桁の数値を表示できます！
#### 接続図
デイジーチェーンを使用する場合はMAX7219同士を信号線で接続します。電源(VccとGND)は並列で接続します。
連結させる数が多い場合はArduinoの電源だけでは十分な電源を供給できないため、外部電源が必要になる場合があります。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/column.png)

#### 設定
1. 基本的にデバイスの設定は通常の7セグメントLEDと同じ流れですが、デバイスをMobiflightボードに追加した際に`Num`をデイジーチェーンで連結されているMAX7219の数を設定します。（今回は２つ連結しているので`2`を設定）
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/column1.png)

1. マッピングを作成する際は`Display`タブの`Display settings`の`Connector`で表示する7セグメントLEDデバイスを指定します。
今回は２つの7セグメントLEDをデイジーチェーン接続しているので１番目と２番目を指定して、それぞれ`Select Preset`にアクティブ/スタンバイの無線周波数を選択します。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/column2.png)
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/column3.png)
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/column4.png)
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/column5.png)
1. デイジーチェーンを使用している場合、デバイスは１つ、マッピングは２つ（デイジーチェーンで接続している数だけ）できます。
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/column6.png)
1. これでアクティブ/スタンバイの無線周波数を3本の信号線だけで表示できました！
![](/images/mobiflight-msfs2020-build-controller/com-radio-7seg-led/column7.png)
