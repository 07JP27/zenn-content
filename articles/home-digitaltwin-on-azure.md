---
title: "Azureで作る「おうちデジタルツイン」"
emoji: "🏡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "iot", "digitaltwin", "switchbot", "azuredigitaltwins"]
published: false
published_at: 2050-12-25 06:00 
---

この記事はAzure Advent Calendar 2022 25日目の記事です。
https://qiita.com/advent-calendar/2022/azure



# はじめに
こんにちは、ガジェットとクラウドが好きな者です。
今回は数年前から頭の中で構想していたスマートホームデバイスを使った「おうちデジタルツイン」をAdvent Calendarに合わせて実装してみました。
私は職業柄Azureをよく使用します。AzureでデジタルツインソリューションといえばAzure Digital Twinsというそのままの名前のサービスがありますが、今回はそれだけではなくデータの吸い上げからデジタルツインを閲覧するUI層までエンドツーエンドでAzureを使って実装してみたいと思います。

## 作ったもの
Webブラウザでデジタルツインの情報を反映した3Dモデルが閲覧できます。
部屋の配置は実際と異なりますが、各機器とデータは実際に私の部屋にある機器からデータを10分間隔で取得して3Dモデル空間に反映しています。
温湿度や、加湿器の水不足アラートなど、各機器に応じて情報が表示されているのがわかると思います。全体の表示モードを切り替えてサイバー感溢れるワイヤーフレーム表示も可能です。
![](/images/home-digitaltwin-on-azure/product-sample.gif)
gif化した都合でカクカクしていますが、実際の画面はもっとヌルヌル動いています。
<!-- textlint-disable ja-technical-writing/no-mix-dearu-desumasu -->
<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->
## この記事は何であって何ではないか
である
- Azureの各種サービスを使用して、個人レベルでもエンドツーエンドのデジタルツインソリューションが構築できます！という事例紹介とキー要素の紹介

ではない
- 上記を構築するための手順やガイド（もしかしたら今後別途書くかも）
<!-- textlint-enable ja-technical-writing/no-mix-dearu-desumasu -->
<!-- textlint-enable ja-technical-writing/ja-no-mixed-period -->

# アーキテクチャ
全体的なデータや操作の流れはこのような形です。（細かくてすみません、拡大してごらんください...）
画像には主にセンサーデータのトランザクショナルなデータの流れと３Dモデルデータなどのスタティックなデータの流れが記載されています。
![](/images/home-digitaltwin-on-azure/architecture.png)

# 各要素のポイント
## エッジデバイス/ゲートウェイ
スマートホームデバイスの雄ことSwitchBot。スマートフォンからボタンを押したり温室度を計ってくれたり、加湿をしてくれたりなど様々なスマートホーム化が可能です。
そして、SwitchBotにはWeb APIが用意されていて2022年10月にVer1.1が公開されています。今回はこのAPIを利用してデバイスの情報を取得しています。
なお、このAPIは今回利用するようなデバイスの情報取得するケースだけでなく、プラグをOn/Offするなどデバイスの操作(コマンド)も可能です。
APIの利用方法について以下のリポジトリが公式ドキュメントになっています。
https://github.com/OpenWonderLabs/SwitchBotAPI

## Azure Digital Twins
Azure Digital Twinsの構築の流れは以下の通りです。
1. モデル定義
1. ツインの生成/ツイングラフの作成
1. データイングレス(ツインに対してデータの更新)

### モデル定義
モデルはオブジェクト指向プログラミングで言うクラスのようなもので、そのデバイスが持っているプロパティなどを定義したデジタルツイン化の対象を抽象したテンプレートのようなものです。

モデルはDigital Twin Definition Language (DTDL)という言語を使用して定義します。
[言語リファレンスはGitHubで公開](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md)されています。実際にコーディングを行う際はVisual Studio Codeの拡張機能である[DTDL Editor](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-dtdl)を使用するとモデルのテンプレート生成やIntellisenseが使えて便利です。

![](/images/home-digitaltwin-on-azure/dtdl.png)

今回作成したモデルファイルは[こちら](https://github.com/07JP27/HomeDigitalTwin/tree/main/ADTModels)で公開しています。

モデルのADTへのアップロードはSDKやREST APIを使用できますが、WebアプリベースのGUI管理ツールである[Azure Digital Twins Explorer](https://learn.microsoft.com/ja-jp/azure/digital-twins/quickstart-azure-digital-twins-explorer)を使用するのが最も簡単です。このツールはモデルファイルのアップロードだけでなく、この後のツインの生成やツイングラフの作成もできます。

### ツインの生成/ツイングラフの作成
ツインはオブジェクト指向プログラミングで言うインスタンスのようなもので、抽象化されたモデルから実在のデバイスに対応するものとして具現化したものです。
さらに、ツインをリレーションシップでつなげることでツイングラフを構築します。

これらの操作についてもSDKやREST APIと共にAure Digital Twins Explorerを使用できます。
以下の画像はAure Digital Twins Explorerで今回のツイングラフを作成している様子です。実際にデジタルツインを構築するのは１部屋だけですが、今後の拡張性を考えてトップレベルのツインを家全体(MyHouse)として各フロアも追加しています。

![](/images/home-digitaltwin-on-azure/twin-graph.png)

### データイングレス
SwitchBot APIとAzure Digital Twinsの通信はAzure Functionsを使って行います。Azure FunctionsはAzureのサーバーレス環境で、イベントドリブンな実行が可能です。10分に1回処理を実行などのタイマー実行もできるので今回のようなケースにはちょうどいいです。実行した回数だけ従量課金というのもコストメリットがあっていいですね。
Azure FunctionsからAzure Digital Twinへの通信はマネージドIDを使用して認証します。Azureサービス間であればマネージドIDを使用することで認証情報などを管理しなくて良くなるので開発者はコーディングに集中でき、メンテナンスフリーというのも嬉しいポイントです。

フルのコードはこちらに載せておきますが、ADTへのデータの投入は以下のような形になります。
```dotnet:InjestToADT
var cred = new DefaultAzureCredential();
var client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred);
var updateTwinData = new Azure.JsonPatchDocument();
updateTwinData.AppendReplace("/LackWater", true);
updateTwinData.AppendReplace("/PowerOn", true);
await client.UpdateDigitalTwinAsync("Humidifier1", updateTwinData);
```

## 3Dモデリング
Blenderでクライアントアプリで表示する3Dモデルをモデリングします。数年ぶりにBlenderを触ったので勉強し直しながらモデリングしましたが、[こちらのYoutube動画シリーズ](https://www.youtube.com/playlist?list=PLbZhON61ML0O75hh8ieSZv95IOYpqSHY_)がわかりやすく学習コンテンツとしてお勧めでした。
モデリングが終わったらファイルをエクスポートしますが、後述の3D Scene studioで使用できるモデル形式はGLTFまたは.GLB形式のため、今回はGLB形式でエクスポートします。BlenderはデフォルトでGLB形式でエクスポートできるようになっています。
![](/images/home-digitaltwin-on-azure/blender-modeling.png)

## クライアントアプリケーション
### 3D Scene studio（まだプレビュー）
今回の肝と言っても過言ではない3D Scene studioですが、最近リリースされたツールで本記事の執筆時点ではまだプレビュー段階です。
Blenderで作成した3DモデルをアップロードしてAzure Digital Twinのツイン/プロパティと3Dモデルの要素とを紐付けをしていきます。方法については[このチュートリアル](https://learn.microsoft.com/ja-jp/azure/digital-twins/quickstart-3d-scenes-studio)を試せば大体わかります。
3D Scene studioでは大きく分けて２つの要素をセットアップします。
- Element：3Dモデルのオブジェクトとデジタルツインの紐付け
- Behavior：表示(色分け/メーター/アイコン)の定義と、その表示とElementとの紐付け

![](/images/home-digitaltwin-on-azure/3d-scene-studio.png)

3D Scene studioは3Dシーンの構築だけでなくビューアーモードも用意されているので、今回はこれを簡易的なクライアントとして使用します。詳しくは後述しますが、この可視化した3DシーンをReactアプリとして独自のアプリに埋め込むことができます。

# 今回は実装しなかったが、今後使えそうな機能
## 3D Scene studioシーンの独自アプリへの埋め込み
3D Scene studioはReactのコンポーネントが提供されているのでそれを使用してWebアプリを作成可能です。
しかし、今回はプレビューということもあり制約などが多く、要件としては3D Scene studioのViewモードで足りるため独自アプリへの埋め込みは実装しませんでした。Webアプリでコントロールパネルを独自に作成してその操作のフィードバックを3Dシーンで確認するまで最終的にはいきたいです...！

なお、ライブラリは以下のリポジトリで公開されており、実際の埋め込み方法はWikiに記載されています。（が、Azure ADのアプリ登録が必要だったり、お世辞にも簡単&わかりやすいとは言えません...GAに期待！）
https://github.com/microsoft/iot-cardboard-js/wiki/Embedding-3D-Scenes

## イベントルートの利用
イベントルートを使用するとツインの更新を別のツインに対して通知できます。
例えば温湿度計ツインのプロパティが更新された際に、その温湿度計が存在している部屋ツインに対して温湿度を複製し、部屋全体の温湿度として扱うことができるようになります。
これにより現実世界をより正確に反映したモデルが実現できます。
![](/images/home-digitaltwin-on-azure/event-route.png)

具体的な仕組みとしては以下の通りEvent HubとAzure Functionsの組み合わせで実現できます。
![](/images/home-digitaltwin-on-azure/routing-workflow.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/digital-twins/concepts-route-events)
https://learn.microsoft.com/ja-jp/azure/digital-twins/concepts-route-events


## Digital Twins データの履歴化
Azure Digital Twins(のツイングラフ)は最新のプロパティを保持しますが、これを履歴化して時系列グラフで表示したい、という要望もあると思います。そこで利用できるのがAzure Data Explorer統合を利用したデータの履歴化です。これにより、Azure Digital TwinsインスタンスをAzure Data Explorerクラスターに接続し、デジタルツインのプロパティ更新をAzure Data Explorerに自動的に履歴化できます。
![](/images/home-digitaltwin-on-azure/data-history-architecture.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/digital-twins/concepts-data-history)
https://learn.microsoft.com/ja-jp/azure/digital-twins/concepts-data-history

この方法を使用すると時系列データがAzure Data Explorerに保存されるので、そこからKustoクエリを使用して時系列データを取得できます。下の画像はAzure Portal上でクエリをかけている様子ですが、[Azure Data Explorer REST API](https://learn.microsoft.com/ja-jp/azure/data-explorer/kusto/api/rest/)を使用して自分のアプリでデータ使用も可能です。
![](/images/home-digitaltwin-on-azure/data-history-run-query-2.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/digital-twins/how-to-use-data-history)

https://learn.microsoft.com/ja-jp/azure/digital-twins/how-to-use-data-history

## SwitchBot Webhook の利用
SwitchBotにはAPI V1.1からWebhookが追加されました。これを使えばリアクティブにデバイスの状態変化を検知できます。
しかし、今回は対応していないデバイスが多かったこと、セキュリティ的に不安があったことからWebhookではなくポーリング方式としました。Webhookが整備されてくればそちらを使いたいと思います。
なぜWebhookの採用に至らなかったのか、詳細は以下のスクラップに記載しています。
https://zenn.dev/07jp27/scraps/4f2ff5aceff79d

## モデルファイルのCI/CD
モデルファイルは現実世界の事物を抽象化したものであり、現実世界の「それ」が変わったり更新されるとモデルも更新する必要があります。モデル定義言語のDTDLのスキーマを見るとわかりますが、モデルファイル自身のバージョニングもできるようになっています。
Azure DevOpsやGitHubのpipelineを使用してコード管理をし、更新されたらcontinuous deployment(継続的なデプロイ)をするようにしてCI/CDサイクルを実現することでより堅牢なモデルファイル管理ができると感じています。

モデルファイルのCI/CDについては以下のブログで触れられています。
https://techcommunity.microsoft.com/t5/internet-of-things-blog/model-lifecycle-management-for-azure-digital-twins/ba-p/2305290

# 今後の展望（拡張）
今回はPoCの意味もあり小さいスコープで実装をしてみましたが、十分実用的に使えそうという感想を持ちました。
今後実際に活用していくにあたって、以下のような拡張によってレベルアップを図りたいと思います。

## 家１軒まるごとデジタルツイン化 & デバイス操作
今回は1部屋3デバイスだけのデジタルツインを作成しましたが、今回実装したものを拡張しておうちまるごと１軒分のデジタルツインを作成することももちろん可能です。
また、SwitchBotのAPIを使用してデバイスの操作も可能ですので、デジタルツインで家の状態を把握しながらデバイスの集中管理を行うこともできるでしょう。
将来的には集中管理ダッシュボードのようなものを作って、家１軒分のデバイスを管理するようなソリューションを作りたいと思っています。


## 他のメーカーのIoTデバイス
今回は私が所有している中で最も種類が多いSwitchBotを対象にしましたが、私は電球などはPhilips hueを使用していたり、様々なデバイスが存在しています。それらのメーカーのIoTデバイスもすべて含めたおうちデジタルツインを作成することで、家の状態の正確な把握と細部までの操作性を実現したいです。

## アプリ常駐化
Amazon Echo Show 15のようなデバイスをリビングルームに設置してそこにアプリを常駐させておくことで集中管理ができます。
一番滞在時間の長い部屋で家全体の状態を把握しながら集中操作をできることができれば、今から行こうとしている部屋の暖房をつけておくなど便利な使い方ができるでしょう。~~あと単純に見た目がかっこいい。~~
![](/images/home-digitaltwin-on-azure/echo-show.png)
画像はイメージです。[引用元](https://www.amazon.co.jp/Echo-Show-15-%E3%82%A8%E3%82%B3%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%BC15-%E3%82%B9%E3%83%9E%E3%83%BC%E3%83%88%E3%83%87%E3%82%A3%E3%82%B9%E3%83%97%E3%83%AC%E3%82%A4-with-Alexa/dp/B08MQNJC9Z)

# 運用コスト
今回構築したソリューションで課金が発生するポイント(サービス)と課金単位は以下の通りです。
- Azure Functions
    実行回数ごとの課金
- Azure Digital Twins
    操作・メッセージ・クエリごとの課金

ここでAzure Digital Twinsのメッセージ・操作・クエリの考え方が曲者です。
たとえば「操作」という単位はAPI経由でのリクエスト1回を1操作と数えるのではなく、ペイロードの容量も1KB単位で考慮されたりします(１回のリクエストでも2KBなら2操作としてカウント)。
そのため正直言ってこの値を正確に予想することはほぼ不可能です。PoCを行う場合はAzure Portalから実際のメトリックが確認できるのでここら辺の情報から推定するのが現実的でしょう。
![](/images/home-digitaltwin-on-azure/adt-metric.png)


今回のケースでの月額料金を[Azure料金計算ツール](https://azure.microsoft.com/ja-jp/pricing/calculator/)で見積もると以下のようになります。
(10分に1回更新でリソースの配置リージョンは東日本場合)
|サービス|課金単位|使用量|料金|
|:--|:--|:--|--:|
|Azure Functions|¥27.759 / 100万実行回数|4,464回|¥27.759|
|Azure Digital Twins|￥346.99 / 100万操作|100万操作以下|￥346.99|
|Azure Digital Twins|￥138.79 / 100万メッセージ|今回は使用しない|¥0|
|Azure Digital Twins|￥69.40 / 100万クエリ|100万クエリ以下|￥69.40|

**合計：￥583 / 月**

意外と安い...! 仮に今後スコープを家全体に範囲を広げたり、独自アプリをホストすることを考えてもそこまで跳ね上がることはないでしょう。(AzureでWebアプリをホストできる[Web Apps](https://azure.microsoft.com/ja-jp/products/app-service/web/)や[Static Web Apps](https://azure.microsoft.com/ja-jp/products/app-service/static/#overview)などには無料プランもあります。)

え？費用対効果？ロマンですよ、ロマン。

# 最後に
いかがでしたでしょうか。なにに使うの？と聞かれればぐうの音も出ませんが、個人開発レベルでも簡単にエンドツーエンドのデジタルツインソリューションが構築できる様子をご覧いただけたと思います。
今はまだリリースしたてで安定しない部分やプレビューなどで今回実現できなかった部分もありますが、2023年も様々なアップデートがあることを期待しつつ、今後もAzure Digital Twinsを使ったソリューションの開発を続けていきたいと思います。

# 参考
今回のおうちデジタルツインを作るにあたって調査したAzure Digital Twinsやその周辺情報は以下のスクラップにまとめています。
https://zenn.dev/07jp27/scraps/d7548193837c70

今回のソリューションを構築するために作成したファイルやプログラムのリポジトリ
https://github.com/07JP27/HomeDigitalTwin
