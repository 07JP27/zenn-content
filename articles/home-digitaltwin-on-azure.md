---
title: "Azureで作る「おうちデジタルツイン」"
emoji: "🏡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "iot", "digitaltwin", "switchbot"]
published: false
---

この記事は Azure Advent Calendar 2022 の 25日目の記事です。
https://qiita.com/advent-calendar/2022/azure



# はじめに
こんにちは、ガジェットとクラウドが好きな者です。
今回は数年前から頭の中で構想していたスマートホームデバイスを使った「おうちデジタルツイン」をAdvent Calendarに合わせて実装してみました。
私は職業柄Azureをよく使用します。AzureでデジタルツインソリューションといえばAzure Digital Twinsというそのままの名前のサービスがありますが、今回はそれだけではなくデータの吸い上げからデジタルツインを閲覧するUI層までエンドツーエンドでAzureを使って実装していきます。
## 作ったもの

## この記事は何であって何ではないか
である
- Azureの各種サービスを使用して、個人レベルでもエンドツーエンドのデジタルツインソリューションが構築できます！という紹介

ではない
- 上記を構築するための手順やガイド（もしかしたら今後別途書くかも）

# アーキテクチャ
全体的なデータや操作の流れはこのような形です。（細かくてすみません、拡大してごらんください・・・）
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
Azure Digital Twinsの構築の流れは
モデル定義→ツインの生成→ツイングラフの構築→ツインに対してデータの更新
となります。
### モデル定義
モデルはオブジェクト指向プログラミングで言うクラスのようなもので、プロパティなど定義したデバイスなどのデジタルツイン化の対象を抽象したテンプレートのようなものです。
### ツイン/ツイングラフ作成
ツインはオブジェクト指向プログラミングで言うインスタンスのようなもので、抽象化されたモデルから実在のデバイスのツインとして具現化したものです。
ツインをリレーションシップでつなげることでツイングラフを構築することができます。
### データイングレス

## 3Dモデリング
Blenderでクライアントアプリで表示する3Dモデルをモデリングします。数年ぶりにBlenderを触ったので勉強し直しながらモデリングしましたが、[こちらのYoutube動画シリーズ](https://www.youtube.com/playlist?list=PLbZhON61ML0O75hh8ieSZv95IOYpqSHY_)がわかりやすく学習コンテンツとしてお勧めでした。
モデリングが終わったらファイルをエクスポートしますが、後述の3D Scene studioで使用できるモデル形式はGLTFまたは.GLB形式のため、今回はGLB形式でエクスポートします。BlenderはデフォルトでGLB形式でエクスポートできるようになっています。
![](/images/home-digitaltwin-on-azure/blender-modeling.png)

## クライアントアプリケーション
### 3D Scene studio（まだプレビュー）
今回の肝になるツールと言っても過言ではない3D Scene studioですが、最近リリースされたツールでまだプレビュー段階です。
Blenderで作成した3DモデルをアップロードしてAzure Digital Twinのプロパティと紐付けをしていきます。方法については[このチュートリアル](https://learn.microsoft.com/ja-jp/azure/digital-twins/quickstart-3d-scenes-studio)を試せば大体わかります。

3D Sceneが完成したらそれをWebアプリに埋め込みます。3D Scene studioはReactのコンポーネントが提供されているのでそれを使用してWebアプリを作成します。
### Webアプリの開発

### 認証設定
Easy Auth


# 今回は実装しなかったが、今後使えそうな機能
## イベントルートの利用
イベントルートを使用するとツインの更新を別のツインに対して通知することができます。
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

この方法を使用すると時系列データがAzure Data Explorerに保存されるので、そこからKustoクエリを使用して時系列データを取得することができます。下の画像はAzure Portal上でクエリをかけている様子ですが、[Azure Data Explorer REST API](https://learn.microsoft.com/ja-jp/azure/data-explorer/kusto/api/rest/)を使用して自分のアプリでデータを使用することも可能です。
![](/images/home-digitaltwin-on-azure/data-history-run-query-2.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/digital-twins/how-to-use-data-history)

https://learn.microsoft.com/ja-jp/azure/digital-twins/how-to-use-data-history

## SwitchBot Webhook の利用
SwitchBotにはAPI V1.1からWebhookが追加されました。これを使えばリアクティブにデバイスの状態変化を検知できます。
しかし、今回は対応していないデバイスが多かったこと、セキュリティ的に不安があったことからWebhookではなくポーリング方式としました。Webhookが整備されてくればそちらを使いたいと思います。
なぜWebhookの採用に至らなかったのはか以下のスクラップで記載しています。
https://zenn.dev/07jp27/scraps/4f2ff5aceff79d

## モデルファイルのCI/CD

# 今後の展望（拡張）
今回はPoCの意味もあり小さいスコープで実装をしてみましたが、十分実用的に使えそうという感想を持ちました。
今後実際に活用していくにあたって、以下のような拡張によってレベルアップを図りたいと思います。
## 家１軒まるごとデジタルツイン化 & デバイス操作
今回は1部屋だけのデジタルツインを作成しましたが、今回実装したものを元に、お家まるごと１軒分のデジタルツインを作成することも可能です。
また、SwitchBotのAPIを使用してデバイスの操作も可能ですので、デジタルツインで家の状態を把握しながらデバイスの集中管理を行うこともできるでしょう。
将来的にはセントラルコントロールボードを作って、家１軒分のデバイスを管理するようなソリューションを作りたいと思っています。

## 他のメーカーのIoTデバイス
今回は私が所有している中で最も種類が多いSwitchBotを対象にしましたが、電球などはPhilips hueを使用していたりと、様々なデバイスが存在しています。
それらのメーカーのIoTデバイスもすべて含めたおうちデジタルツインを作成することで、家の状態の正確な把握と細部までの操作性を実現したいです。

## アプリ常駐化
Amazon Echo Show 15のようなデバイスをリビングルームに設置してそこにアプリを常駐させておくことで、一番滞在時間の長い部屋で家全体の状態を常に把握しながら集中操作をできることが最終的な理想形です。

# 費用見積もり

費用対効果？ロマンですよ、ロマン。

# 参考
今回のおうちデジタルツインを作るにあたって調査したAzure Digital Twinsやその周辺情報は以下のスクラップにまとめています。
https://zenn.dev/07jp27/scraps/d7548193837c70
