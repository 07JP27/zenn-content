---
title: "【実践編】無線周波数操作パネルを作る"
---
# 本チャプターの内容
いよいよ今まで１つずつ習得してきた要素をまとめて１つの機能として実装します。
今回は航空管制無線に使用される周波数を操作するパネルを作ります。
### 完成イメージ
https://www.youtube.com/watch?v=y3pY7OdbyVE

# 準備するもの
- [Arduino UNO R3](https://akizukidenshi.com/catalog/g/gM-07385/)
- [Arduino UNO用ユニバーサル基板](https://www.amazon.co.jp/gp/product/B07DRHG1QV)
- [7セグメントLED (MAX7219チップ)](https://www.amazon.co.jp/gp/product/B088CR8LR6)
- [ロータリーエンコーダー](https://akizukidenshi.com/catalog/g/gP-06357/)
- [パネル取り付けスイッチ](https://akizukidenshi.com/catalog/g/gP-15668/)
- [ワイヤー(AWG24)](https://akizukidenshi.com/catalog/g/gP-11090/)
- [熱収縮チューブ](https://akizukidenshi.com/catalog/g/gP-06788/)
- [ピンヘッダ](https://akizukidenshi.com/catalog/g/gC-00167/)
- [タッピングビス(呼び径3)](https://www.amazon.co.jp/s?k=%E3%82%BF%E3%83%83%E3%83%94%E3%83%B3%E3%82%B0%E3%83%93%E3%82%B9+3)
- [スズメッキ線](https://akizukidenshi.com/catalog/g/gP-02220/)

![](/images/mobiflight-msfs2020-build-controller/radio-panel/parts2.jpg)
今回は必要なピン数が少ないのでArduino UNO R3を使用しますが、動作が不安定だったり、他にも機能を付けたい場合はArduino Megaを使用します。


# 回路図
このような接続になります。
基本的には各要素で説明したものが独立してそれぞれ接続されているだけです。
7セグメントLEDのみコラムで紹介したデイジーチェーンを使用して接続しています。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/breadboard.png)


# ブレッドボード上でのテスト
いきなりハンダ付けをしてしまうと万が一、回路が間違っていた場合に修正が大変なので実際にブレッドボードで同じ回路を作って試してみます。
とりあえず動作としては良さそうということが確認できたので次に進みます。
https://www.youtube.com/watch?v=GhuZNTBqI2c

# 筐体
回路むき出しでもそれはそれでメカメカしくて良いのですが、せっかくホームコックピットを作るのでそれっぽい外装にします。
Fusion 360を使って筐体をCADで設計します。[Fusion 360](https://www.autodesk.co.jp/products/fusion-360/personal)は個人利用であれば無料で使用できるCADソフトです。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/case1.jpeg =400x)
![](/images/mobiflight-msfs2020-build-controller/radio-panel/case2.png =400x)

作成した3D CADモデルからSTLファイルをエクスポートし、3Dプリンターで部品を作成しました。外装になる部分はエアブラシでそれっぽい色に塗装してあります。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/parts.png)

# 組み立て
フレームを組み立てます。Arduino UNOもマウントに取り付けます。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem1.png)

I/O部品をパネルに取り付けます。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem2.png)
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem3.png)

回路図に従ってどんどん結線をしていきます。このときにケーブルの色を5V(Vcc)は赤、GNDは黒、信号線は白に色分けをするとわかりやすいです。もっと複雑になったら白を細分化して色分けしてもいいかもしれません。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem4.png)
I/Oの部品が付いたパネルをフレームに取り付け、Arduino UNO用のユニバーサル基板にケーブルを接続していきます。
このユニバーサル基板は[一般的なユニバーサル基板](https://akizukidenshi.com/catalog/g/gP-03230/)に比べて割高なのですが、一般的なユニバーサル基板だとArduino UNOのピンのピッチ(穴の間隔)と合わない場合があるので、とても重宝しています。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem5.png)
それぞれの接点は密集しているのでピンヘッダをハンダ付けしてそこにケーブルを接続して熱収縮チューブでカバーしておきます。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem6.png)

ケーブルをまとめます。熱収縮チューブを使ってまとめていく方法もありますが、その方法だと最初からケーブルを熱収縮チューブに通して置かないといけないので、マスキングテープや結束バンドで最後にまとめるほうが簡単です。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem7.png)

サイドパネルやフェイスパネル、ノブを取り付けて完成です！
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem8.png)


# MobiFlightの設定
基本的には今まで学習してきた設定の組み合わせなので一部省略して記載しています。
### デバイスの設定
ボードにはデバイスを３つ登録します。
- 7セグメントLED（デイジーチェーンで1つのデバイスで２つのLEDモジュールを使用します）
- ロータリーエンコーダー
- ボタン
![](/images/mobiflight-msfs2020-build-controller/radio-panel/1.png)
7セグメントLEDの設定です。ピンは実際の接続の通りに設定します。デイジーチェーンを使用して２つの7セグメントLEDを使用するので`Num`を`2`に設定します。
その他のデバイスは実際の接続のピン通りに設定をするだけです。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/2.png)
![](/images/mobiflight-msfs2020-build-controller/radio-panel/3.png)
![](/images/mobiflight-msfs2020-build-controller/radio-panel/4.png)
設定が完了したら忘れずにボードへアップロードをしましょう。
### フライトシミュレーターとデバイスのマッピング
最初に出力マッピングから作成します。必要な出力は以下の２つのマッピングです。
- Active：現在のアクティブな無線周波数を表示する7セグメントLED
- Standby：スタンバイ無線周波数を表示する7セグメントLED
![](/images/mobiflight-msfs2020-build-controller/radio-panel/101.png)
**Activeの設定**
![](/images/mobiflight-msfs2020-build-controller/radio-panel/102.png)
![](/images/mobiflight-msfs2020-build-controller/radio-panel/103.png)
**Standbyの設定**
![](/images/mobiflight-msfs2020-build-controller/radio-panel/104.png)
![](/images/mobiflight-msfs2020-build-controller/radio-panel/105.png)

次に入力マッピングを作成します。必要な入力は以下の２つのマッピングです。
- Freq：スタンバイ無線周波数を変更するロータリーエンコーダー
- Swap：スタンバイとアクティブの無線周波数を入れ替えるボタン
![](/images/mobiflight-msfs2020-build-controller/radio-panel/106.png)
**Freqの設定**
![](/images/mobiflight-msfs2020-build-controller/radio-panel/107.png)
![](/images/mobiflight-msfs2020-build-controller/radio-panel/108.png)
**Swapの設定**
![](/images/mobiflight-msfs2020-build-controller/radio-panel/109.png)

# 実行
1. MSFS2020を起動します。
1. すべてのマッピングのActiveにチェックが入っていることを確認し、MobiFlight Connectorの`Run`ボタンをクリックして接続を開始します。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/201.png)
1. MSFS2020で適当なフリーフライトを開始します。
1. 実際に作成したコントローラーとMSFS2020内の無線パネルが連動して動作することを確認します。

https://www.youtube.com/watch?v=y3pY7OdbyVE

# まとめ
前半で１つずつ習得してきた要素を組み合わせて１つの機能として実装してみました！
このように作成した１機能ごとのコントローラーを並べることでホームコックピットを構成していきます。
次はオートパイロットのパネルを作ろうかなと思っています。

### 改善点
今回、実際に作ってみて以下のような改善点を感じました。
- 周波数変更のノブがkHzノブしか無い【致命的】
  MHz変更できないと意味ないやんけ...
  本物はノブが２重になっていてMHzとkHzを変更できるようなちょっと複雑な機構なので今後はそれをやってみたいと思います。
- スイッチめっちゃ重い
  パネル取り付け用のボタンを使いましたが、そこそこ押し込むのに力がいります。
  もっと軽いものを選ぶか、パネルに取り付けることができるタクトスイッチがあればそれにすると良いでしょう。
- 本物の航空機の配置とは違う
  これは一長一短ですね。リアルな配置では無いと言えばネガティブですが、自分の好きな配置やスペースに合わせて作れるという点では利点になると思います。
