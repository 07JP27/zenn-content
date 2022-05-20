---
title: "スロットルレバーを可変抵抗で制御する"
---
# 本チャプターの内容
このチャプターでは可変抵抗でアナログ値の入力を学びます。アナログ値は0/1のデジタル値とはことなり、より細かいレベルを入力できます。
### 完成イメージ
可変抵抗を回すと抵抗値が変化し、それに伴ってスロットルが増加/減少します。
https://www.youtube.com/watch?v=XjDd-JFavMo

# 準備するもの
- [Arduino](https://akizukidenshi.com/catalog/c/carduino1/)
- [可変抵抗(10〜100kΩ程度)](https://akizukidenshi.com/catalog/g/gP-16468/)
- [ブレッドボード](https://akizukidenshi.com/catalog/g/gP-05294/)
- [ジャンパーワイヤー](https://akizukidenshi.com/catalog/g/gC-05159/)

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
1. デバイスを追加したいMobiFlightボードを選択して`Add device`→`Analog Input`をクリックします。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/1.png)
1. ボードに選択したデバイスが追加されます。接続の通り`Pin settings`を`A0`にセットし、アップロードボタンをクリックしてMobiFlightボードに変更を書き込みます。OKをクリックして`Settings`画面を閉じます。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/2.png)

### フライトシミュレーターとデバイスのマッピング
1. ホーム画面で`Inputs`タブをクリックしてInputsマッピングに切り替えて、`Throttle`マッピングを作成します。edit列の`...`をクリックして`InputConfigWizard`画面を表示します。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/101.png)
1. デバイスを選択して、`THROTTLE_SET`を設定します。この場合、すべてのスロットルが同期して操作されます。双発機などスロットルが複数ある航空機で使用する場合は複数の可変抵抗を用意してそれぞれスロットルレバーごとにマッピングも可能です。
設定ができたら`OK`をクリックして`InputConfigWizard`画面を閉じます。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/102.png)

# 実行
1. MSFS2020を起動します。
1. MSFS2020が起動していればMobiFlight Connectorの`Run`ボタンがクリックできるようになるので、マッピングがActiveになってることを確認して`Run`ボタンをクリックします。
![](/images/mobiflight-msfs2020-build-controller/throttle-potentiometer/201.png)
1. MSFS2020で適当なフリーフライトを開始します。
1. 可変抵抗を回してMSFS2020内のスロットルレバーが連動することを確認します。

https://www.youtube.com/watch?v=XjDd-JFavMo

※可変抵抗を回しやすいように3Dプリンターで作成したノブを付けています。

# まとめ
これでMobiFlightで用意されているすべての入力方法を習得しました！
これまで学んできたデバイスの組み合わせで、コックピットのほぼすべてのアビオニクスを作成できることでしょう！
