---
title: "フラップゲージをサーボモーターで出力する"
---
# 本チャプターの内容
今回はアクチュエーターであるサーボモーターを使ってフラップゲージを駆動させてみましょう。アクチュエーターとは電気信号を物理的運動に変換する機械や機構を指します。
今までは電気信号を電気信号として出力するLEDを使ってあまり動きのない出力をしてきましたが、アクチュエーターを使用することによってよりダイナミックな出力を表現できます。
### 完成イメージ
フラップレベルに連動して、サーボモーターの回転角度が変化します。
https://www.youtube.com/watch?v=tWNil0x5OM0

# 準備するもの
- [Arduino](https://akizukidenshi.com/catalog/c/carduino1/)
- [サーボモーター](https://www.amazon.co.jp/gp/product/B08D3JH2PF)
- [ブレッドボード](https://akizukidenshi.com/catalog/g/gP-05294/)
- [ジャンパーワイヤー](https://akizukidenshi.com/catalog/g/gC-05159/)

# 回路図
Arduinoと準備した電子部品を以下のように接続します。
![](/images/mobiflight-msfs2020-build-controller/flap-servomotor/breadboard.png)
回路図で表すと、このようになります。
![](/images/mobiflight-msfs2020-build-controller/flap-servomotor/circuit.png =600x)
サーボモーターの駆動はPWM制御と呼ばれる、パルス波のデューティー比を変化させて変調して制御する方法が使われます。([参考サイト](https://monoist.itmedia.co.jp/mn/articles/0706/06/news132.html))
**ArduinoにはPWMに対応しているピンと対応していないピンがあります。ピン番号の前に`~`と記載があるピンがPWM対応のピン**なのでサーボモーターを接続する場合はPWM対応のピンを選択してください。
![](/images/mobiflight-msfs2020-build-controller/flap-servomotor/pwm.png)

:::message
今回の接続ではサーボモーターの電源はArduinoから供給していますが、サーボモーターを複数接続するなどArduinoから十分な電源を供給できない場合は外部電源を接続してください。([参考サイト](https://takawo.hatenablog.com/entry/2017/08/22/193251))
:::


# MobiFlightの設定
### デバイスの設定
1. MobiFlight Connectorを起動してメニューバーから `Extras`→`Settings`をクリックして`Settings`画面を開きます。`MobiFlight Modules`タブを開くと現在接続されているMobiFlightボードがリスト表示されます。
1. デバイスを追加したいMobiFlightボードを選択して`Add device`→`Servo`をクリックします。
![](/images/mobiflight-msfs2020-build-controller/flap-servomotor/1.png)
1. ボードに選択したデバイスが追加されます。接続の通り`DIN line`を`6`にセットし、アップロードボタンをクリックしてMobiFlightボードに変更を書き込みます。OKをクリックして`Settings`画面を閉じます。
![](/images/mobiflight-msfs2020-build-controller/flap-servomotor/2.png)
### フライトシミュレーターとデバイスのマッピング
1. 新規Outputを作成して`Flap`と名前をつけてedit列の`...`をクリックします。
1. ConfigWizardが表示されます。`Sim Variable`タブで`Select Preset`を`FLAPS HANDLE INDEX` を選択します。
![](/images/mobiflight-msfs2020-build-controller/flap-servomotor/101.png)
1.  ConfigWizardの`Display`タブに移動して以下の設定をします。
今回はフラップレベルを５段階で出力するのでMin valueとMax valueを0~4の5段階に設定します。
`Test`をクリックして駆動するか確認します。正常に駆動したら`OK`をクリックしてConfigWizardを閉じます。
![](/images/mobiflight-msfs2020-build-controller/flap-servomotor/102.png)

# 実行
1. MSFS2020を起動します。
1. MSFS2020が起動していればMobiFlight Connectorの`Run`ボタンがクリックできるようになるので、FlapのマッピングがActiveになってることを確認して`Run`ボタンをクリックします。
![](/images/mobiflight-msfs2020-build-controller/flap-servomotor/201.png)
1. MSFS2020で適当なフリーフライトを開始します。
1. 画面内でフラップを操作し、それに連動してサーボモーターが駆動することを確認します。(以下の動画ではキーボードでフラップを操作しています)

https://www.youtube.com/watch?v=tWNil0x5OM0

# まとめ
コックピットでは物理的運動を使用する操作は多くはありませんが、これらもちゃんと表現できるとより一層リアルなコックピットに近づくことでしょう。
次のチャプターではいよいよこれまで習得してきた各要素を組み合わせて１つのコンポーネントを作ってみます。
