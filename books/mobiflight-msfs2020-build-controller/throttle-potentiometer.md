---
title: "[WIP]スロットルを可変抵抗で制御する"
---

:::message alert
このチャプターは執筆途中です。
:::

# 本チャプターの内容

# 準備するもの
- Arduino
- 可変抵抗(10〜100kΩ程度)
- ブレッドボード
- ジャンパーワイヤー

# 可変抵抗について
可変抵抗（ポテンショメーター）は音楽のボリューム調整などに使用されることが多い部品です。
主に２つの種類があり、回転運動によって抵抗値を変化させるロータリーポテンショメーターと、直線運動によって抵抗値を変化させるリニアポテンショメーターがあります。
航空機の入力として使用されることが多いのはロータリーポテンショメーターですが、いずれも動作原理は同じで分圧回路によって抵抗値を変化させることで電圧を上げ下げしてアナログ入力をします。
- ロータリーポテンショメーター([出典：秋月電子](https://akizukidenshi.com/catalog/g/gP-15812/))
![](https://akizukidenshi.com/img/goods/C/P-15812.jpg =300x)

- リニアポテンショメーター([出典：秋月電子](https://akizukidenshi.com/catalog/g/gP-09238/))
![](https://akizukidenshi.com/img/goods/3/P-09238.jpg =300x)

# 回路図
Arduinoと準備した電子部品を以下のように接続します。今までは0/1のデジタル入力でしたが、今回はアナログ入力になるためArduinoボードのアナログ対応ピン（ピン番号にAとついてるピン）に信号線を接続します。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/breadboard.png =400x)
回路図で表すと、このようになります。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/circuit.png =600x)

# MobiFlightの設定
### デバイスの設定
1. MobiFlight Connectorを起動してメニューバーから `Extras`→`Settings`をクリックして`Settings`画面を開きます。`MobiFlight Modules`タブを開くと現在接続されているMobiFlightボードがリスト表示されます。
1. デバイスを追加したいMobiFlightボードを選択して`Add device`→`encorder`をクリックします。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/1.png)
1. ボードに選択したデバイスが追加されます。接続の通り`Left Pin`を`2`、`Right Pin`を`3`にセットし、アップロードボタンをクリックしてMobiFlightボードに変更を書き込みます。OKをクリックして`Settings`画面を閉じます。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/2.png)

### フライトシミュレーターとデバイスのマッピング
1. ホーム画面で`Inputs`タブをクリックしてInputsマッピングに切り替えて、`COM Radio freq`マッピングを作成します。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/101.png)
1. `COM Radio freq`マッピングのedit列の`...`をクリックして`InputConfigWizard`画面を表示します。
1. ロータリーエンコーダーの場合、時計回りと反時計周りでそれぞれ動作を設定します。
`On Left`は反時計回りを意味するので無線周波数を減少させる`COM_RADIO_KHZ_DEC_ENCODER`を設定します。ここで`COM_RADIO_MHZ_DEC_ENCODER`にするとMHz単位で変更できるようになります。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/102.png)
1. `On Right`は時計回りを意味するので無線周波数を増加させる`COM_RADIO_KHZ_INC_ENCODER`を設定します。
２つの設定ができたら`OK`をクリックして`InputConfigWizard`画面を閉じます。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/103.png)

# 実行
1. MSFS2020を起動します。
1. MSFS2020が起動していればMobiFlight Connectorの`Run`ボタンがクリックできるようになるので、マッピングがActiveになってることを確認して`Run`ボタンをクリックします。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/201.png)
1. MSFS2020で適当なフリーフライトを開始します。
1. 可変抵抗を回してMSFS2020内のスロットルレバーが連動することを確認します。
※ロータリーエンコーダーを回しやすいように3Dプリンターで作成したノブを付けています。

# まとめ
これでMobiFlightで用意されているすべての入力方法を習得しました！
これまで学んできたデバイスの組み合わせで、ほぼすべてのコックピットのアビオニクスを作成できることでしょう！
