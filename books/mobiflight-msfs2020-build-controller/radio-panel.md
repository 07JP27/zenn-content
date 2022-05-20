---
title: "【実践編】無線周波数操作パネルを作る"
---
# 本チャプターの内容
いよいよ今まで１つづつ試してきた要素をまとめて１つの機能として実装します。
今回は航空無線に使用される周波数を操作するパネルを作ります。

# 準備するもの
- Arduino UNO R3
- Arduino UNO用ユニバーサル基板
- 7セグメントLED (MAX7219チップ)
- ロータリーエンコーダー
- パネル取り付けスイッチ
- ワイヤー(AWG24)
- 熱収縮チューブ
- ピンヘッダ
- タッピングビス(φ3)
- ニクロム線

![](/images/mobiflight-msfs2020-build-controller/radio-panel/parts2.jpg)


# 回路図
![](/images/mobiflight-msfs2020-build-controller/radio-panel/breadboard.png)


# ブレッドボード上でのテスト
いきなりハンダ付けをしてしまうと万が一回路が間違っていると修正が大変なので実際にブレッドボードで同じ回路を作って試してみます。
とりあえず動作としては良さそうということが確認できたので次に進みます。
https://www.youtube.com/watch?v=GhuZNTBqI2c

# 筐体
回路むき出しでも良いのですが、せっかくホームコックピットを作るのでそれっぽい外装にします。
Fusion 360を使って筐体をCADで設計します。Fusion 360は個人利用であれば無料で使用することができるCADソフトです。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/case1.jpeg =400x)
![](/images/mobiflight-msfs2020-build-controller/radio-panel/case2.png =400x)

作成した3D CADモデルからSTLファイルをエクスポートし、3Dプリンターで部品を作成しました。外装になる部分は塗装してあります。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/parts.png)

# 組み立て
フレームを組み立てます。Arduino UNOもマウントに取り付けます。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem1.png)

I/O部品をパネルに取り付けます。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem2.png)
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem3.png)

回路図に従ってどんどん結線をしていきます。このときにケーブルの色を5V(Vcc)は赤、GNDは黒、信号線は白に色分けをするとわかりやすいです。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem4.png)
I/Oの部品が付いたパネルをフレームに取り付け、Arduino UNO用のユニバーサル基板にピンヘッダをハンダ付けしてケーブルを接続していきます。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem5.png)
それぞれの接点は密集しているので熱収縮チューブでカバーしておきます。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem6.png)

ケーブルをまとめます。熱収縮チューブを使ってまとめていく方法もありますが、その方法だと最初からケーブルを熱収縮チューブに通して置かないといけないので、マスキングテープや結束バンドで最後にまとめるほうが簡単です。
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem7.png)

サイドパネルやフェイルパネルを取り付けて完成です！
![](/images/mobiflight-msfs2020-build-controller/radio-panel/assem8.png)


# MobiFlightの設定
### デバイスの設定
### フライトシミュレーターとデバイスのマッピング

# 実行
ではいつもどおりMSFS2020を起動後にMobiFlight Connectorの`Run`ボタンで接続してフリーフライトで試してみましょう。
https://www.youtube.com/watch?v=y3pY7OdbyVE

# まとめ
というわけで今まで１つづつ試してきた要素をまとめて１機能として実装してみました！
実際に作ってみて以下のような改善点を感じました。
### 改善点
- 周波数変更のノブがkHzノブしか無い【致命的】
  MHz変更できないと意味ないやんけ・・・・
  本物はノブが２重になっていてMHzとkHzを変更できるようなちょっと複雑な機構なので今後はそれをやってみたいと思います。
- スイッチめっちゃ重い
  パネル取り付け用のボタンを使いましたが、そこそこ押し込むのに力がいります。
  もっと軽いものを選ぶか、パネルに取り付けることができるタクトスイッチがあればそれにすると良いでしょう。
- 本物の航空機の配置とは違う
  これは一長一短ですね。リアルな配置では無いと言えばネガティブですが、自分の好きな配置やスペースに合わせて作れるという点では利点になると思います。


次はオートパイロットのパネルを作ろうかなと思っています。




