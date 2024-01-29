---
title: "OpenAIとMicrosoft Graph Search APIでM365の組織内データを検索するRAGアプリを作る"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai", "microsoftgraph"]
published: true
publication_name: "microsoft"
---
共著者：[@maruryupro](https://twitter.com/maruryupro) / [@kazuyan](https://twitter.com/D_kazuyan)

社内ナレッジを検索するRAGアプリを作ろうとする場合、さまざまな文献でAzure AI Search(旧称Cognitive Search)を使うケースを多く目にします。Azure AI Searchは非常に高機能で組織専用の検索エンジンを作ることができる一方、データの前処理やインデクサーのカスタマイズなど様々なテクニックが必要になり、実現したいこととコストのトレードオフが発生するケースが多いように感じます。
この記事では、M365テナント内の情報をWeb API経由で検索できる「Microsoft Search Graph API」を使って、専用のインデックスを作成する必要なく社内の情報を検索してOpenAIを通したチャット形式で返答するRAGアプリの実装アプローチについて紹介します。

![](/images/azure-openai-graph-rag-pattern/chat-sample.png)

# TL;DR サンプルアプリのリポジトリ
とりあえずコードを読みたい方はこちらからどうぞ。
https://github.com/07JP27/azureopenai-internal-microsoft-search

# アプローチ
言葉で説明するよりもまずは登場人物とデータの流れから全体像をイメージしていただく方が早いと思いますので、以下の図をご覧ください。

**全体図**
![](/images/azure-openai-graph-rag-pattern/overview_ja.png)

**シーケンス図**
![](/images/azure-openai-graph-rag-pattern/sequence_ja.png)

## ポイント
### Microsoft Search
Microsoft Search Graph APIの説明をする前に「Microsoft Search」についてご紹介します。
**Microsoft Searchはを一言で説明するとM365テナント内のデータをユーザーに適した形で検索、提供してくれる機能**です。
例えば組織アカウントてログインした状態のBingで何かを検索すると、組織名の検索結果とともに「職場」というタブが表示されます。
![](/images/azure-openai-graph-rag-pattern/bing.png)

この「職場」タブはMicrosoft Searchによって提供されている機能で、以下のようにM365テナント内のデータを検索できます。
（ダミーデータを使っています）
![](/images/azure-openai-graph-rag-pattern/mssearch.png)
![](/images/azure-openai-graph-rag-pattern/ms-search-sample.png)

Bing以外にもWord OnlineやSharePoint Onlineなどさまざまなアプリで検索エリアが提供されており、Microsoft Searchによる検索を行うことができます。
![](/images/azure-openai-graph-rag-pattern/search_bar.png)

#### Microsoft Search Graph API 
そしてここがこの記事最大のポイントですが、**Microsoft SearchはGraph API経由で呼び出すことができます。** つまりMicrosoft Searchの機能を独自のアプリに組み込むことができるというわけです。

**Microsoft Search Graph APIのポイント**
- サービス横断検索：複数種類のデータ（例えばSharePointリストとOneDriveアイテム）を横断で検索することができます。ただし、組み合わせることができるデータの種類は[限られています。]( https://learn.microsoft.com/ja-jp/graph/search-concept-interleaving)
- パーソナライズ：Microsoft Search Graph APIをコールする際にユーザーごとのJWTを使用する場合、その**ユーザーに権限があるファイルのみを関連度に応じて返すことができます。**
- 外部データの取り込み：後述するMicrosoft Graph コネクタを使うことで、M365外のデータを検索対象にすることができます。
- 1 つの統合検索エンドポイント：さまざまな種類のデータを１つのAPIエンドポイントに対するリクエストで検索できます。リクエストボディによって検索する内容を制御します。

**Microsoft Search Graph APIで検索できるデータ**
- chatMessage：Teamsのメッセージ
- message：Exchange Onlineのメール
- event：Exchange Onlineのカレンダーイベント
- drive：SharePoint Onlineのドキュメントライブラリ
- driveItem：SharePoint Onlineのファイル、フォルダ、ページ、ニュース
- list：SharePoint OnlineとOneDriveのリスト
- listItem：SharePoint OnlineとOneDriveのリストアイテム
- site：SharePoint Onlineのサイト
[and more...](https://learn.microsoft.com/ja-jp/graph/api/resources/search-api-overview?view=graph-rest-1.0&preserve-view=true#scope-search-based-on-entity-types)


ここまでの話をまとめるとMicrosoft SearchやMicrosoft Search Graph APIの関係は以下のようなイメージになります。
![](/images/azure-openai-graph-rag-pattern/hierarchy.png)



## Azure　AI Searchとの比較
冒頭にも述べたように組織ないのナレッジRAGアプリを作る場合、データの検索にはAzure AI Searchを使うケースを多くみかけます。ではAzure AI Searchでデータソースをインデックス化し、それを検索に使用する場合とMicrosoft Search Graph APIを使用する場合ではどのような違いがあるのでしょうか？簡単に以下の表にまとめてみました。

| 比較項目 | Microsoft Search Graph API | AI Search |
| --- | --- | --- |
| 情報の鮮度 | ほぼリアルタイム | インデクサーが実行された時点 |
| 権限を考慮した検索 | ユーザーごとのトークンを使うと検索の際に自動フィルターされる | [セキュリティフィルター](https://learn.microsoft.com/ja-jp/azure/search/search-security-trimming-for-azure-search)を反映してインデックスを作成するように構成・実装する必要がある |
| 検索精度 | ブラックボックス | 全文検索/セマンティック検索/ハイブリッド |
| カスタマイズ性 | 検索処理の部分はカスタマイズ不可 | インデクサーのスキルやアナライザーなど細かいカスタマイズが可能 |
| ベクトル検索 | 不可 | 可 |
| コスト発生箇所 | Webアプリ | AI Search,Webアプリ、インデックス作成 |


## 考慮事項
ほとんどのアプローチは銀の弾丸ではないように、このアプローチもすべての社内検索シナリオに最適なわけではありません。以下にいくつかの考慮事項を挙げます。
#### 検索範囲を考える
テナント内をまるっと検索できることは非常に汎用的である反面、検索タスクに適用するシナリオが膨大になってしまう懸念があります。
例えばすでに用意されている出張規定の中から旅費規程を見つけるよりも、社内に膨大に存在するドキュメントの中から旅費規程を探す方が難易度が高いのは想像が付きます。
このことは2023年11月に開催されたOpenAI DevDayの中でもRAGのプラクティスとして[「シグナル/ノイズ比」という考え方で紹介されています](https://youtu.be/ahnGLM-RC1Y?si=-VFIoYLiMfPt3AYz&t=1286)。
例えば業務範囲や検索範囲がすでに絞ることができる場合は検索対象をそれらのリストやライブラリに限定することで検索精度の向上が期待できます。
Microsoft Search APIでは[特定のサイトやデータの種類を制限して検索することもできます](https://learn.microsoft.com/ja-jp/graph/search-concept-files#example-5-use-filters-in-search-queries)。

#### 文章形式と文章量を考える
元データをインデックス化する際に整形するAI Searchを使用した方法と異なり、このアプローチは元データをそのまま検索します。つまりAI Serachにインデックス化をする際に行う画像のOCRや音声の文字起こし、文章の整形などの処理を行うことができません。これが意味することは、このアプローチは元データがテキストデータで最も効果を発揮するということです。
また、元データの文章量も考慮する必要があります。AI Searchを使用した方法ではデータソースをLLMに入力することができる上限を超えないようにデータソースを分割する「チャンキング」が行われますが、このアプローチでは元データをそのまま使用するため、元データの文章量が多い場合は回答の生成ができない、または回答生成の精度が低下する可能性があります。


# よくありそうなQA
## 精度はどうなの？
当たり前といえば当たり前ですが、専用に設計されたAI Searchの方が体感的に良いです。
上記の「Azure　AI Searchとの比較」からもわかる通り、このアプローチのメリットは精度を多少犠牲にして構築/運用コストを下げることができるという点です。
また、適用範囲(業務)・データ形式によっても精度は大きく変わります。

### 参考：RAGアーキテクチャにおける精度影響箇所
RAGアーキテクチャにおいて「精度」に影響する箇所は大きく3点があります。
- クエリ生成精度：入力文→検索クエリの生成。多くの場合はLLMが担う。
- 検索精度：クエリに対して検索エンジンが返す結果の精度。今回のアプローチの場合はMicrosoft Search (Graph API)が担う。
- 回答生成精度：検索されてきた結果を元に回答を生成する精度。多くの場合はLLMが担う。

ここでクエリ生成と回答生成はLLMが担うためプロンプトエンジニアリングの世界になります。つまり自前開発をする場合はプロンプトを工夫することでなんとでもなります。
よってこのシステムを評価する上で最も重要な点はブラックボックス化されているMicrosoft Search Graph APIの検索精度ですが、これは前述したとおり、組織アカウントでBingにログインしていると使用することができるMicrosoft Searchと同じものなので、まずはBingのMicrosoft Searchを使って検索精度を試してみることをお勧めします。



## まるっと検索とは言っても検索対象にできるのってM365内のデータだけなんでしょ？
これは半分正解で半分不正解です。Microsoft Graphには**Microsoft Graph コネクタという仕組みがあり、外部のサードパーティのデータをMicrosoft Search 結果に表示することができます。**
これだけを聞くと不正解のように見えますが、実は外部のサードパーティのデータを検索対象にする過程でインデックス化という処理を実行してM365側にインデックスを取り込みます。よって検索対象のインデックスはM365内のデータとなるため正解であり不正解ということになります。
Microsoft Graphコネクタは「Microsoft提供」と「パートナー提供」の２種類があり、要するに前者がMicrosoft純正となり、Azure DevOpsやJira Cloud、ServiceNowのコネクタがあります。
すべてのMicrosoft提供コネクタとパートナー提供のコネクタの一覧は「[Microsoft Graph コネクタ ギャラリー](https://www.microsoft.com/microsoft-search/connectors/?category=&query=)」で確認できます。また独自のコネクタを使用することも可能です。

**参考**
- [Microsoft Search API を使用してデータのインデックスを作成する](https://learn.microsoft.com/ja-jp/graph/api/resources/indexing-api-overview?view=graph-rest-1.0)
- [Microsoft Search API を使用してデータをクエリする(externalItem)](https://learn.microsoft.com/ja-jp/graph/api/resources/search-api-overview?view=graph-rest-1.0)

## それってMicrosoft365 Copilotと同じじゃないの？
機能だけを見た場合、結果的にそうなる可能性が高いです。しかしM365 CopilotはSaaS形式なのに対して、このサンプルはPaaSを利用しているため開発の余地が皆様にあり、**Your Copilotにこの機能を組み込む**ことができるという点は大きな違いです。具体的には開発によって以下のような部分の制御が可能になります。

* スコープの設定：特定のサイトや特定のドライブのみを検索対象にするなど、ユーザーの権限の中でさらに情報を取得する範囲をさらに制御する。
* プロンプト：検索のためのクエリを生成したり、回答を生成するためのプロンプトを開発者が完全に制御できる。
* その他のデータソースとの連携：Microsoft Graph APIだけではなくてその他のサービスのレスポンスも組み合わせた回答を生成する

## Graph APIの利用に追加費用はかかるの？
基本的にはかかりません。＝M365のユーザーライセンスに内包されています。今回紹介したアプリを実装する場合は追加費用なしで実装可能です。
https://learn.microsoft.com/ja-jp/graph/metered-api-overview

ただし、Teamsチャットのエクスポートなど一部の操作は従量課金の支払いモデルが発生します。
https://learn.microsoft.com/ja-jp/graph/metered-api-list


# サンプルアプリ
上記のアプローチを実装したサンプルアプリを公開しています。
#### 機能・特徴
- Microsoft 365内のドキュメントやサイト、Teamsの投稿などをデータソースにしたLLMによるチャット形式の内部ナレッジ検索
- Microsoft 365でも使用されるMicrosoft Search APIを使用したシンプルなRAGアーキテクチャ
- [On-Behalf-Of フロー](https://learn.microsoft.com/ja-jp/entra/identity-platform/v2-oauth2-on-behalf-of-flow)によるユーザーの権限に応じた検索

https://github.com/07JP27/azureopenai-internal-microsoft-search

リポジトリへのPull Requestも歓迎です。

# 利用に最適なシナリオ
個人的には以下のような理由からTeamsのチャットをデータソースとして回答したい場合はこのアプローチが最適なんじゃないかと考えています。
- ユーザーごとに権限を考慮した検索が可能
- リアルタイムに近い情報の検索
- テキストデータの検索が最適

# 【追記】精度と権限管理を両立させるハイブリッド方式の提唱
記事の公開以来、本記事に関するお問い合わせを多くいただいております。その多くはやはり権限管理と精度を両立する方法が無いか知りたいというものです。以下に筆者が考えたハイブリッド方式をアイディアとして記載します。
※ここから先の情報はあくまでアイディアベースでまだ実装して試してはいません。

今回の方式のメリットである
- 権限管理を反映したナレッジを検索できる

とAI Searchを使った方式のメリットである
- 検索精度が高い（ベクター検索・セマンティックランキング）
- チャンク分割に対応しているためピンポイントな検索が可能

という２つの利点を両立させる方法を考えた結果、次の２つ(+派生型)のハイブリッド方式が有効ではないかと考えています。
ただし、いずれの場合でもGraph APIとAI Searchを両方用意する必要があるので、今回の方式の「簡単に」というメリットはなくなります。

## Option1. Graph 先行方式
### 流れ
1. チャンク分割をした状態で、AI Searchに対象検索データを全てインデックス化します。この時にインデックスのカスタムフィールドにファイルを一意に識別できるIDまたはファイルパスなどを保持します。
1. 今回の方式と同様にOBOフローでGraph APIを呼び出し、権限を考慮した検索結果を取得します。
1. Graphの検索結果から帰ってきたファイルを一意に識別できるIDまたはファイルパスを検索条件に入れて、再度AI Searchに検索をかけます。
1. 指定した権限を持つファイルのみをチャンク分割した状態でAI Searchの結果から取得できます。
1. AI Searchから帰ってきた情報を使って最終回答を生成します。

### メリット
- 外部サービスへのアクセス回数が少ない＝高速

### デメリット
- 最初の検索精度がGraph APIの検索精度に依存する（Graphで取れてこないものはそもそもAI Searchの検索条件に含まれない）

![](/images/azure-openai-graph-rag-pattern/hybrid1.png)
![](/images/azure-openai-graph-rag-pattern/hybrid1_seq.png)


## Option2. AI Search 先行方式
### 流れ
1. チャンク分割をした状態で、AI Searchに対象検索データを全てインデックス化します。この時にインデックスのカスタムフィールドにファイルを一意に識別できるIDまたはファイルパスなどを保持します。
1. AI Searchに対してクエリを発行して検索結果を取得します。（この時点では権限が考慮されていない）
1. AI Searchから帰ってきた結果から元ファイルを特定します。
1. 特定したファイルに対して、ユーザーごとのトークンでアクセスを試行します。
1. アクセスできれば権限あり、アクセスできない場合は権限なし、or削除済みと判定します。
1. アクセスできた情報のみをコンテキストとして最終回答を生成します。

### メリット
- AI Searchの高い検索精度を実現

### デメリット
- Graph API呼び出しが複数回発生するため速度やスロットリングに懸念がある
![](/images/azure-openai-graph-rag-pattern/hybrid2.png)
![](/images/azure-openai-graph-rag-pattern/hybrid2_seq.png)

## Option3. Option2の派生型
Option2でGraph APIを使ってアクセス試行をする際、ドライブアイテムの[アクセス許可の取得API](https://learn.microsoft.com/ja-jp/graph/api/permission-get?view=graph-rest-1.0&tabs=http)を使ってアクセス権限を取得し、ログインユーザーIDと突合するような方式も考えられます。その場合はユーザーIDだけ分かればOKなので、そもそもOBOフローが不要になる可能性があります。