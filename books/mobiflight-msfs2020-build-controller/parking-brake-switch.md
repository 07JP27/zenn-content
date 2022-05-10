---
title: "[WIP]パーキングブレーキをタクトスイッチでON/OFFする"
---

:::message alert
このチャプターは執筆途中です。
:::

# 本チャプターの内容
前チャプターではLEDを使ったシンプルな出力をしました。
このチャプターではパーキングブレーキの状態をタクトスイッチで切り替えることを題材に、シンプルな入力を学びます。
前チャプターで出力部分を作成したので、それをそのまま使用して入力を追加します。前チャプターをやっていなくても問題はありません。
具体的な動作イメージはタクトスイッチ１を押すとパーキングブレーキがON(LEDもON)になり、タクトスイッチ２を押すとパーキングブレーキがOFF(LEDもOFF)になるような動作です。

# 準備するもの
- Arduino
- LED
- 抵抗(330~1kΩ程度)
- タクトスイッチ ２個
- ブレッドボード
- ジャンパーワイヤー

# 回路図
Arduinoと準備した電子部品を以下のように接続します。前チャプターで接続したLEDをそのまま使用して追加でパーキングブレーキのON/OFF用タクトスイッチを２つ接続します。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/breadboard.png =400x)
回路図で表すと、このようになります。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/circuit.png =700x)

この回路図で電子工作をしている人はわかると思いますが、タクトスイッチは[プルアップ](https://miraiworks.org/?p=6213#i)で接続をしています。MobiFlightではArduinoに付属している内部プルアップ抵抗を使用するため外部プロアップ抵抗は必要ありません。(出典：[公式フォーラム](https://www.mobiflight.com/forum/topic/2888.html))

# MobiFlightの設定
### デバイスの設定
1. MobiFlight Connectorを起動してメニューバーから `Extras`→`Settings`をクリックして`Settings`画面を開きます。`MobiFlight Modules`タブを開くと現在接続されているMobiFlightボードがリスト表示されます。
1. デバイスを追加したいMobiFlightボードを選択して`Add device`→`Button`をクリックします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/1.png)
1. ボードに選択したデバイスが追加されます。`Pin settings`を`2`にセットします。今回はスイッチを２つ接続するので判別できるように`Name`を`Brake On`に設定します。
1. `Add device`→`Button`を繰り返してもう１つのスイッチを追加します。`Pin settings`を`4`、`Name`は`Brake Off`に設定します。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/2.png)
1. アップロードボタンをクリックしてMobiFlightボードに変更を書き込みます。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/3.png)
1. アップロード完了画面が出るまで待ちます。アップロードが完了したら`OK`をクリックしてSettings画面を閉じ、ホーム画面に戻ります。

### フライトシミュレーターとデバイスのマッピング
1. ホーム画面で`Inputs`タブをクリックしてInputsマッピングに切り替えます。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/101.png)
1. 「Double-click row to ~」と記載がある部分をダブルクリックして`Brake On`と入力します。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/102.png)
1. 同様に新規マッピングを追加して`Brake Off`と入力します。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/103.png)
1. `Brake On`マッピングのedit列の`...`をクリックします。
1. `InputConfigWizard`画面が表示されます。`Input`タブでの`Choose input`セクションでマッピングするデバイスを選択します。`On Press`タブで`Action Type`を`MSFS2020`に、`Select Preset`を`PARKING_BRAKES_ON`を選択します。
通常Select Presetの選択肢は膨大にあるので`Filter Preset List`の検索条件を使用して機種で絞り込んだり、フリーワード検索をします。
設定ができたら`OK`をクリックして`InputConfigWizard`画面を閉じます。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/104.png)
1. 同様に`PARKING_BRAKES_OFF`の設定をします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/105.png)
1. ホーム画面に戻ると以下のような画面で各列が設定されていることを確認します。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/106.png)
1. マッピングを作成した2つの`Active`にチェックを入れます。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/107.png)

# 実行
1. MSFS2020を起動します。
1. MSFS2020が起動していればMobiFlight ConnectorのRunボタンがクリックできるようになるので、Runボタンをクリックします。
![](/images/mobiflight-msfs2020-build-controller/parking-brake-switch/201.png)
1. MSFS2020で適当なフリーフライトを開始します。
1. ON/OFFボタンを押してMSFS2020のパーキングブレーキが連動して動作すること、LEDがある場合はそれも連動して点灯/消灯することを確認します。

https://www.youtube.com/watch?v=6tybPh-r-j4


# まとめ
これでシンプルな出力と入力ができるようになりました。
入力ができることによって、ホームコックピット感が大幅に増したと思います！
次のチャプターでは、より複雑な出力として無線周波数表示やオートパイロットの表示に使われる7セグメントLEDを使用した出力をしてみます。


# コラム
### 実行のタイミングを変えたい