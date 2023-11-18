---
title: "OpenAIとMicrosoft Graph Search APIでM365の組織内データを検索するRAGアプリを作る"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai", "microsoftgraph"]
published: false
publication_name: "microsoft"
---
共著者：[@maruryupro](https://twitter.com/maruryupro)

社内ナレッジを検索するRAGアプリを作ろうとする場合、さまざまな文献でAzure Cognitive Searchを使うケースを多く目にします。Azure Cognitive Searchは非常に高機能で組織専用の検索エンジンを作ることができる一方、データの前処理やインデクサーのカスタマイズなど様々なテクニックが必要になり、実現したいこととコストのトレードオフが発生するケースが多いように感じています。
この記事では、M365テナント内の情報をWeb API経由で検索できる「Microsoft Search Graph API」を使って、社内の情報を検索してOpenAIを通したチャット形式で返答するRAGアプリについて紹介します。

# TL;DR サンプルアプリのリポジトリ
とりあえずコードを読みたい方はこちらからどうぞ！
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



## Azure　Cognitive Searchとの比較
冒頭にも述べたように組織ないのナレッジRAGアプリを作る場合、データの検索にはAzure Cognitve Searchを使うケースを多くみかけます。ではAzure Cognitve Searchでデータソースをインデックス化し、それを検索に使用する場合とMicrosoft Search Graph APIを使用する場合ではどのような違いがあるのでしょうか？簡単に以下の表にまとめてみました。

| 比較項目 | Microsoft Search Graph API | Cognitive Search |
| --- | --- | --- |
| 情報の鮮度 | ほぼリアルタイム | インデクサーが実行された時点 |
| 権限を考慮した検索 | ユーザーごとのトークンを使うと検索の際に自動フィルターされる | [セキュリティフィルター](https://learn.microsoft.com/ja-jp/azure/search/search-security-trimming-for-azure-search)を使ってインデックスを作成するように構成・実装する必要がある |
| 検索精度 | ブラックボックス | 全文検索/セマンティック検索/ハイブリッド |
| カスタマイズ性 | カスタマイズできる部分が少ない | インデクサーのスキルやアナライザーなど細かいカスタマイズが可能 |
| ベクトル検索 | 不可 | 可 |


# よくありそうなQA
## 精度はどうなの？
精度と一口に言っても様々な指標がありますが、ここでは「望んだ回答が返ってくる&その回答が誤ってない」という若干ふわっとした定義をします。（実際に本番適用のための評価であれば「[再現率](https://predictionone.sony.biz/cloud_manual/terminology/recall/)​」や「[正解率](https://predictionone.sony.biz/cloud_manual/terminology/accuracy/index.html)」など定量的な指標を使うことをお勧めします。）
RAGアーキテクチャにおいて「精度」に影響する箇所は大きく3点があります。
- クエリ生成精度：入力文→検索クエリの生成。多くの場合はLLMが担う。
- 検索精度：クエリに対して検索エンジンが返す結果の精度。今回のアプローチの場合はMicrosoft Search (Graph API)が担う。
- 回答生成精度：検索されてきた結果を元に回答を生成する精度。多くの場合はLLMが担う。

ここでクエリ生成と回答生成はLLMが担うためプロンプトエンジニアリングの世界になります。つまり自前開発をする場合はプロンプトを工夫することでなんとでもなります。
よってこのシステムを評価する上で最も重要な点はブラックボックス化されているMicrosoft Search Graph APIの検索精度ですが、これは前述したとおり、組織アカウントでBingにログインしていると使用することができるMicrosoft Searchと同じものなので、まずはBingのMicrosoft Searchを使って検索精度を試してみることをお勧めします。
完全に主観にはなりますがMicrosoft Search単体でもMicrosoft Search Graph APIを組み込んだサンプルアプリでも特に精度が悪いとは感じませんでした。特にMicrosoft Searchは私も普通に仕事で使っており検索精度には、満足しています。


## まるっと検索とは言っても検索対象にできるのってM365内のデータだけなんでしょ？
これは半分正解で半分不正解です。Microsoft Graphには**Microsoft Graph コネクタという仕組みがあり、外部のサードパーティのデータをMicrosoft Search 結果に表示することができます。**
これだけを聞くと不正解のように見えますが、実は外部のサードパーティのデータを検索対象にする過程でインデックス化という処理を実行してM365側にインデックスを取り込みます。よって検索対象のインデックスはM365内のデータとなるため正解であり不正解ということになります。
Microsoft Graphコネクタは「Microsoft提供」と「パートナー提供」の２種類があり、要するに前者がMicrosoft純正となり、Azure DevOpsやJira Cloud、ServiceNowのコネクタがあります。
すべてのMicrosoft提供コネクタとパートナー提供のコネクタの一覧は「[Microsoft Graph コネクタ ギャラリー](https://www.microsoft.com/microsoft-search/connectors/?category=&query=)」で確認できます。また独自のコネクタを使用することも可能です。

**参考**
- [Microsoft Search API を使用してデータのインデックスを作成する](https://learn.microsoft.com/ja-jp/graph/api/resources/indexing-api-overview?view=graph-rest-1.0)
- [Microsoft Search API を使用してデータをクエリする(externalItem)](https://learn.microsoft.com/ja-jp/graph/api/resources/search-api-overview?view=graph-rest-1.0)

## それってMicrosoft365 Copilotと同じじゃないの？
機能だけを見た場合、結果的にそうなる可能性が高いです。しかしM365 CopilotはSaaS形式なのに対して、このサンプルはPaaSを利用しているため開発の余地が皆様にあり、**Your Copilotにこの機能を組み込む**ことができるという点は大きな違いです。具体的には開発によって以下のような部分の制御が可能になります。(なお本記事執筆時点でM365 Copilotはリリースされていないのであくまで想像である点にご注意ください)

* スコープの設定：特定のサイトや特定のドライブのみを検索対象にするなど、ユーザーの権限の中でさらに情報を取得する範囲をさらに制御する。
* プロンプト：検索のためのクエリを生成したり、回答を生成するためのプロンプトを開発者が完全に制御できる。
* その他のデータソースとの連携：Microsoft Graph APIだけではなくてその他のサービスのレスポンスも組み合わせた回答を生成する

## Graph APIの利用に追加費用はかかる？
基本的にはかかりません。＝M365のユーザーライセンスに内包されています。今回紹介したアプリを実装する場合は追加費用なしで実装可能です。
https://learn.microsoft.com/ja-jp/graph/metered-api-overview

ただし、Teamsチャットのエクスポートなど一部の操作は従量課金の支払いモデルが発生します。
https://learn.microsoft.com/ja-jp/graph/metered-api-list


# サンプルアプリ
上記のアプローチを実装したサンプルアプリを公開しています。
#### 機能・特徴
- Microsoft 365内のドキュメントやサイト、Teamsの投稿などを基にしたLLMによるチャット形式の内部ナレッジ検索
- Microsoft 365でも使用されるMicrosoft Search APIを使用したシンプルかつ高精度なRAGアーキテクチャ
- [On-Behalf-Of フロー](https://learn.microsoft.com/ja-jp/entra/identity-platform/v2-oauth2-on-behalf-of-flow)によるユーザーの権限に応じた検索

https://github.com/07JP27/azureopenai-internal-microsoft-search

リポジトリへのPull Requestも歓迎です。