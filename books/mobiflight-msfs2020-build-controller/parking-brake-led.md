---
title: "パーキングブレーキの状態をLEDで表示する"
---
# 本チャプターの内容
最初の一歩として、LEDを使ったシンプルな出力から始めましょう。
このチャプターではパーキングブレーキの状態をLEDで表示することを題材に、シンプルな二値(ON/OFF)の出力を学びます。
パーキングブレーキがONの時はLEDは点灯し、OFFの時は消灯するような動作です。

# 準備するもの
- Arduino
- LED
- 抵抗(330~1kΩ程度)
- ブレッドボード
- ジャンパーワイヤー

# 回路図
Arduinoと準備した電子部品を以下のように接続します。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/breadboard.png =400x)
回路図で表すと、このようになります。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/circuit.png =700x)

# MobiFlightの設定
最初のチャプターなのでMobiFlightの使い方も含めてスクリーンショット多めで解説します。

MobiFlightでは大きく分けて２つ設定を行います。
1. デバイスの設定
MobiFlightではLEDやスイッチなどの電子部品を「デバイス」と呼びます。
MobiFlightの設定では最初にArduinoにどのデバイスがどのように(どのピンに)接続されているか」を設定します。
1. フライトシミュレーターとデバイスのマッピング
デバイスの設定が完了したら、そのデバイスにフライトシミュレーター内のどの項目（今回はパーキングブレーキの状態）を表示するかをマッピングします。

この２つによって、**どのようなデバイスがArduinoに接続されており**、そのデバイスに**フライトシミュレーター内のどのような情報を出入力するか**を設定します。それでは具体的に設定を行っていきましょう。

### デバイスの設定
1. MobiFlight Connectorを起動してメニューバーから `Extras`→`Settings`をクリックします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/1.png)
1. `Settings`画面が表示されます。`MobiFlight Modules`タブを開くと現在接続されているMobiFlightボードがリスト表示されます。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/2.png)
1. デバイスを追加したいMobiFlightボードを選択して`Add device`→`LED/Output`をクリックします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/3.png)
1. ボードに選択したデバイスが追加されます。今回は8番ピンにLEDを接続するので`Pin settings`を`8`にセットします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/4.png)
1. デバイスの変更を行ったら必ずボードに変更を書き込みます。ボードリスト下のアップロードアイコンをクリックしてデバイス設定をアップロードします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/5.png)
1. アップロード完了画面が出るまで待ちます。アップロードが完了したら`OK`をクリックしてSettings画面を閉じ、ホーム画面に戻ります。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/6.png)

### フライトシミュレーターとデバイスのマッピング
1. ホーム画面で「Double-click row to ~」と記載がある部分をダブルクリックして`Parking brake`と入力します。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/101.png)
1. `Parking brake`マッピングが作成されたので、edit列の`...`をクリックします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/102.png)
1. ConfigWizardが表示されます。`Sim Variable`タブで`Select Preset`を`BRAKE PARKING INDICATOR` を選択します。
通常Select Presetの選択肢は膨大にあるので`Filter Preset List`の検索条件を使用して機種で絞り込んだり、フリーワード検索をします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/103.png)
1. ConfigWizardの`Display`タブに移動して以下の設定をします。
`Test`をクリックすると接続したLEDが点灯します。正常に点灯したら`OK`をクリックしてConfigWizardを閉じます。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/104.png)
1. ホーム画面でマッピングを作成した`Parking brake`の`Active`にチェックを入れてして`Test`ボタンをクリックします。
LEDが画面の表示に合わせて点滅します。これでMobiFlightの設定は完了です。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/105.png)

# 実行
1. MSFS2020を起動します。
1. MSFS2020が起動していればMobiFlight ConnectorのRunボタンがクリックできるようになるので、Runボタンをクリックします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/201.png)
1. MSFS2020で適当なフリーフライトを開始します。
1. 飛行機のパーキングブレーキを画面上でON/OFFし、LEDの点灯/消灯が連動することを確認します。

https://www.youtube.com/watch?v=W96cWgCxOTs

# まとめ
これでON/OFFの二値をLEDで表示できました。
最もシンプルな出力でしたが、今回のパーキングブレーキだけではなくギアの状態やストロボ灯インジケーターなど様々な部分に応用可能です！

# コラム
### 出力を反転させたいときは？
今回はパーキングブレーキがONのとき、LEDもONになるような動作をしました。
ではパーキングブレーキがONのときにLEDをOFFに、パーキングブレーキがOFFのときにLEDをONにするにはどうすればよいでしょう。
この設定を追加するにはマッピングを設定した`ConfigWizard`画面の`Compare`タブで`Comparation Settings`を設定します。
具体的には以下の画像の通り、パーキングブレーキの状態(current value)が1(ON)ならLEDへの出力を0(OFF)、それ以外のときはLEDの出力を1(ON)になるようにします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-led/column.png)
