---
title: "無線周波数をロータリーエンコーダーで入力する"
---

# 本チャプターの内容
前チャプターでは7セグメントLEDでの数値出力を行いました。
このチャプターでは無線周波数をダイヤル（ロータリーエンコーダー）で設定することを題材に、ロータリーエンコーダーの入力を学びます。前チャプターで7セグメントLEDを使用した無線周波数の出力部分を作成したので、それをそのまま使用して入力を追加します。前チャプターをやっていなくても問題はありません。

具体的な動作イメージはロータリーエンコーダーを時計回りに回すとスタンバイの無線周波数が増加し、反時計回りに回すと減少するような挙動です。
前チャプターの7セグメントLEDが接続されていればスタンバイの無線周波数がロータリーエンコーダーの操作に連動して表示されます。
通常の航空機ではノブが２重になっていてKHzとMHzの両方を変更できるようなインターフェイスになっていますが、今回は説明を簡単にするために１つのみにしました。

# 準備するもの
- Arduino
- ロータリーエンコーダー
- ブレッドボード
- ジャンパーワイヤー

# 回路図
Arduinoと準備した電子部品を以下のように接続します。
![](/images/mobiflight-msfs2020-build-controller/com-radio-rotary-encoder/breadboard.png =400x)
回路図で表すと、このようになります。
![](/images/mobiflight-msfs2020-build-controller/com-radio-rotary-encoder/circuit.png =700x)

# MobiFlightの設定
### デバイスの設定
1. MobiFlight Connectorを起動してメニューバーから `Extras`→`Settings`をクリックして`Settings`画面を開きます。`MobiFlight Modules`タブを開くと現在接続されているMobiFlightボードがリスト表示されます。
1. デバイスを追加したいMobiFlightボードを選択して`Add device`→`encorder`をクリックします。
![](/images/mobiflight-msfs2020-build-controller/com-radio-rotary-encoder/1.png)
1. ボードに選択したデバイスが追加されます。接続の通り`Left Pin`を`2`、`Right Pin`を`3`にセットし、アップロードボタンをクリックしてMobiFlightボードに変更を書き込みます。OKをクリックして`Settings`画面を閉じます。
![](/images/mobiflight-msfs2020-build-controller/com-radio-rotary-encoder/2.png)

### フライトシミュレーターとデバイスのマッピング
1. ホーム画面で`Inputs`タブをクリックしてInputsマッピングに切り替えて、`COM Radio freq`マッピングを作成します。
![](/images/mobiflight-msfs2020-build-controller/com-radio-rotary-encoder/101.png)
1. `COM Radio freq`マッピングのedit列の`...`をクリックして`InputConfigWizard`画面を表示します。
1. ロータリーエンコーダーの場合、時計回りと反時計周りでそれぞれ動作を設定します。
`On Left`は反時計回りを意味するので無線周波数を減少させる`COM_RADIO_KHZ_DEC_ENCODER`を設定します。ここで`COM_RADIO_MHZ_DEC_ENCODER`にするとMHz単位で変更できるようになります。
![](/images/mobiflight-msfs2020-build-controller/com-radio-rotary-encoder/102.png)
1. `On Right`は時計回りを意味するので無線周波数を増加させる`COM_RADIO_KHZ_INC_ENCODER`を設定します。
２つの設定ができたら`OK`をクリックして`InputConfigWizard`画面を閉じます。
![](/images/mobiflight-msfs2020-build-controller/com-radio-rotary-encoder/103.png)

# 実行
1. MSFS2020を起動します。
1. MSFS2020が起動していればMobiFlight Connectorの`Run`ボタンがクリックできるようになるので、マッピングがActiveになってることを確認して`Run`ボタンをクリックします。
![](/images/mobiflight-msfs2020-build-controller/com-radio-rotary-encoder/201.png)
1. MSFS2020で適当なフリーフライトを開始します。
1. ロータリーエンコーダーを回してMSFS2020内のスタンバイ無線周波数が変更できること、前チャプターで作成した7セグメントLEDが接続されている場合は、それも連動して表示されることを確認します。

https://www.youtube.com/watch?v=OZRRuR6Dq8Y
※ロータリーエンコーダーを回しやすいように3Dプリンターで作成したノブを付けています。

# まとめ
これでロータリーエンコーダーを使用した入力ができるようになりました。
ここまで学んできたLED、スイッチ、7セグメントLED、ロータリーエンコーダーを組み合わせることで、すでに数多くのコックピット要素を作成できるようになっているはずです。
次のチャプターでは、スロットルなどに使用される可変抵抗（ポテンショメーター）を使ったアナログ値の入力を行います。

# クリーンアップ
使用しているピンが多くなってきたので、次のチャプターへ進む前に今回設定したデバイスとマッピングを削除します。
設定をこのまま残したい場合は削除する必要はありません。
