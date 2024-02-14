---
title: "Azure OpenAI Serviceの日本語記事まとめ"
emoji: "🌏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'openai']
published: true
publication_name: "microsoft"
---

**Azure OpenAI Service**についての日本語記事のまとめです。主に**公式ドキュメント以外の**ブログやZenn/Qiitaの記事をまとめています。ボリュームが多いので、目次から気になる項目を選択してご覧ください。
※長く使える知見のまとめにしたかったので一過性のニュース的な記事や内容が重複している機能紹介記事などは意図的に掲載していません。


**この記事は[GitHub](https://github.com/07JP27/zenn-content/blob/2dc0249a7e74ba8ced2ac4355b063dbfb8b05c0f/articles/azure-openai-japanese-blogs.md)で管理されています。まとめへの追加修正はプルリクエストまたはIssuesでお気軽にお寄せください！**

また、以前に本記事をご覧いただき、そこからの差分を知りたい場合は[GitHubのHistory](https://github.com/07JP27/zenn-content/commits/main/articles/azure-openai-japanese-blogs.md)も併せてご覧いただけると把握しやすいかと思います。

# 概要
## まずはここから
#### Azure OpenAI Service を使い始める
Azure OpenAI Serviceの概要から実際のリソースデプロイ、プレイグラウンドとAPIでの呼び出しまでを解説した記事です。Azure OpenAI Serviceをどのように準備してどのように使うのか、概略を掴むのに最適な記事です。
https://zenn.dev/microsoft/articles/1a15305021cd01

#### Azure OpenAI Service の最初の一歩： Azure版 ChatGPT を Azure OpenAI Studio のチャット用プレイグラウンドで試す
Azure OpenAI Serviceの概要から実際のリソースデプロイ、プレイグラウンドでの利用を解説した記事です。特にプレイグラウンドツールのAzure OpenAI Studioについて詳しく解説しています。
https://qiita.com/youtoy/items/ff8248b2a6778038bd3c

#### ChatGPT - Azure OpenAI 大全
Azure OpenAI ServiceについてX上で数多くの情報発信をされているHirosato Gamoさん渾身のChatGPTとAzure OpenAIサービス全体について纏められた132ページの資料です。
https://speakerdeck.com/hirosatogamo/chatgpt-azure-openai-da-quan

#### Azure OpenAI Serviceのラーニングパスまとめメモ
Microsoftの無料EラーニングプラットフォームMicrosoft Learnで提供されているAzure OpenAI Serviceについての学習コンテンツをまとめた記事です。公式教材で体系的に学びたい人におすすめです。
https://qiita.com/k-kimino/items/7a55ac364201b8da228e

#### 生成 AI アプリケーションの開発を始めるために必要な全知識を学べる 12 講座
様々なLLMの基礎から主にAzure OpenAIに関することが学べる無料のラーニングパスです。
日本語もサポートしていて、Microsoftでクラウド・アドボケイトを務める[寺田佳央](https://twitter.com/yoshioterada)さんが日本語訳をされています。
非常に充実した内容で、Azure OpenAIのハンズオンやサンプルなどコンテンツがあふれるほど含まれています。
https://microsoft.github.io/generative-ai-for-beginners/#/translations/ja-jp/

## 本家OpenAI と Azure OpenAIの違い
#### [比較表] Azure OpenAIと本家OpenAI APIの比較表
とりあえずこれを知っておけばOKという違いを見やすい表形式でまとめた記事です。
https://zenn.dev/microsoft/articles/e0419765f7079a

#### 本家 OpenAI と Azure OpenAI Service の ChatGPT API の比較
利用規約から価格やAPIインターフェースまで幅広く詳細に比較が記載されています。
https://zenn.dev/microsoft/articles/59448047cd5ed9

#### CTOの視点から見たAzure OpenAI ServiceとOpenAIのChatGPT APIの深堀り比較
現役CTOの視点から、性能、セキュリティ、料金、サポートなどの要素について詳しく分析し、自社でのアーキテクチャ選定と使い分けを紹介しています。自社への導入を考えている方は必見です。
https://qiita.com/lazy-kz/items/32e8e7c86bdce67beb48

#### Azure OpenAI Service と OpenAI 本家で同じモデルに同じプロンプトを投げて時間はかってみた
タイトルの通りです。　この記事内では言及されていませんが、本家OpenAIと異なりAzure OpenAI Serviceにはリソースの「リージョン」の考え方があるので、利用者から近いリージョンにデプロイすることで本家OpenAI以上に返答速度が速くなる可能性があります。
https://zenn.dev/microsoft/articles/openai-performance

## API
#### Azure OpenAIの最新APIスキーマを確認する方法
https://zenn.dev/microsoft/articles/azure-openai-latest-api-schema

#### Azure OpenAI に REST API を使用して質問するサンプルコードの作成方法
アプリを作るときはライブラリやSDKを使うことが多いと思いますが、REST APIでの呼び出しはサービスとのインターフェイスを理解する上で大切です。
この記事で紹介されているAPI呼び出しはAzureポータルのAzure CLIから実行する方法のため、ローカルの開発環境がなくても試せます。
https://cptechweb.teldevice.co.jp/hc/ja/articles/20253365966233-Azure-OpenAI-%E3%81%AB-REST-API-%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%A6%E8%B3%AA%E5%95%8F%E3%81%99%E3%82%8B%E3%82%B5%E3%83%B3%E3%83%97%E3%83%AB%E3%82%B3%E3%83%BC%E3%83%89%E3%81%AE%E4%BD%9C%E6%88%90%E6%96%B9%E6%B3%95

#### Azure OpenAIでJSON Modeを使う！
https://zenn.dev/microsoft/articles/azure-openai-jsom-mode

#### Azure OpenAI Service の使用中に発生する可能性がある一般的なエラー コードについて
日本マイクロソフトのサポートチームブログです。
Azure OpenAI Serviceの使用中に発生する可能性がある一般的なエラーコードの内容や原因、解決策について記載されています。
https://jpaiblog.github.io/blog/2023/12/29/AzureOpenAIService-error-codes/

## プロンプトエンジニアリング
#### [メモ] 責任あるAIのためのプロンプトエンジニアリング
https://zenn.dev/microsoft/articles/3a946a22bc6f76

### 各種Prompt Engineeringの日本語実例集（Zero-CoT、mock、ReAct、ToT、Metacog、Step Back、IEPなど）
https://qiita.com/YutaroOgawa2/items/aca32f8fd7d551596cf8

#### Azure Open AI Service で SQL を生成するプロンプト例
https://zenn.dev/masakikato/articles/azure-openai-prompt-sql

## Embeddings
#### Azure OpenAI Embedding モデルを利用し最も関連性の高いドキュメントを見つける方法
https://qiita.com/yoshioterada/items/3e575828368bf3767532

#### 【AzureOpenAI】Embedding(埋め込み化)で熊本のおいしいお店を探してみた
https://qiita.com/kkawano_neko/items/c6c47d8bb6838b187f7b

#### Azure OpenAI Service と Azure Cache for Redis でベクトル検索を行う
https://zenn.dev/microsoft/articles/22d03aa3b2462c

#### Cognitive Searchの生成AI用ベクトルDBの構築手順書
https://qiita.com/tmiyata25/items/875f563ba7a91f3da823

#### 【Golang】Azure OpenAI で Embedding したベクトルを使って、自前検索エンジンを作ろう
https://zenn.dev/microsoft/articles/20231208_go_aoai_pg_vector

## Fine-tuning
#### Azure OpenAI ServiceのChatGPTモデルでのFine-Tuningを試してみた
https://blog.jbs.co.jp/entry/2023/10/24/110804

# リソース管理
## 全般
#### Azure Open AIのコーポレートガバナンスについて考える
Azure OpenAI Serviceで扱われるデータの種類・処理・保持について整理した記事です。Azure OpenAI Serviceに対してリクエストしたデータがどのような流れを経て処理されるのがを理解するのに役立ちます。
https://zenn.dev/microsoft/articles/fe09382d4e35ba

## デプロイ
#### Azure OpenAI ServiceリソースをTerraformで作成する
コードとしてのインフラストラクチャ（IaC:Infrastucture as Code）のための人気ツールであるTerraformを使ってAzure OpenAI Serviceのリソースをデプロイする方法が記載されています。
https://qiita.com/AsukaSeki/items/551f49888e36d6dbdf0b

## 認証
#### Azure OpenAI ServiceのRBACについてまとめた
ここもAzure特有の機能ですが、Azure共通の認証基盤であるAzure ADを使ったRole Based Access Control(RBAC)についての記事です。Azure OpenAI Serviceのリソースに対してAzure ADを使用して権限を管理する方法が記載されています。
https://qiita.com/Yosuke_Sakaue/items/2273bb3e64e0905ac462

#### Azure OpenAI Service でマネージド ID を使う
ユーザーだけでなくリソース間でもAzure AD認証を使用することができます。この記事ではApp Serviceに設定したマネージドIDをコードから参照してAzure OpenAI Serviceを使用する方法が記載されています。
https://zenn.dev/microsoft/articles/openai-managedid

#### Azure OpenAI ServiceのAPIキーを無効化する
特に何も考えずにAzure OpenAI Serviceをデプロイするとそのリソース固有のAPIキーが自動生成されますが、セキュリティ的には使用できるのであればマネージドID/Azure AD認証を使用することが推奨です。
この記事ではAPIキーの無効化する方法について記載されています。
https://techblog.ap-com.co.jp/entry/2023/08/14/140000

#### Azure OpenAI ServiceでAzure AD認証を使ってみる (Python)
https://zenn.dev/microsoft/articles/9d33c143df589e

## クォーター
#### Azure OpenAI Service の クォータ管理
2023年6月まではモデルのデプロイごとにクオーターが固定だったのですが、アップデートによって全体の枠の中でユーザーが各デプロイに上限を割り振れるようになりました。その機能についての説明です。
https://zenn.dev/microsoft/articles/be24a299f46a4d

#### Azure OpenAI Service の Chat Completion API を JMeter で負荷テストしてみた
実際問題どれくらい使ったらクオーター制限に引っ掛かるの？という実験記事です。Azrure OpenAI Serviceはスロットリングウィンドウが１分でリセットされるため、１リクエストあたりのリクエストトークンを増やしてタスクを複雑にし、生成に時間をかけるようにすることでその間にスロットリングウィンドウがリセットされる場合があるというのが興味深いです。
https://zenn.dev/microsoft/articles/20231011_aoai_jmeter

## モデル/バージョン
#### Azure OpenAI Service で GPT-4 API を使う
GPTの次世代モデルであるGPT-4について利用法が記載されています。APIを使うとなっていますが、GPT-4の利用申請方法やPythonのSDKを使った呼び出しなども記載されています。
https://zenn.dev/microsoft/articles/c2cfdda734b61c

#### Azure OpenAI Service のレガシーモデルと今後のモデル選択戦略
Azure OpenAI Serviceってリリースから半年くらいしか経ってないのにもうレガシー扱いのモデルが出てきてるんですよね・・・。というわけで今後モデルはどう選んでいったらいいのかという内容の記事です。
https://zenn.dev/microsoft/articles/b050574ad7dfe2

#### Azure OpenAI Service の gpt-35-turbo と gpt-4 の 2023 年 6 月 バージョン (0613)
0613というバージョンのgpt-35-turboとgpt-4モデルが新しくリリースされました。前バージョンである0301との比較を各項目で行っています。
https://zenn.dev/microsoft/articles/ed503d31efb434

#### OpenAIのモデルアップデートに備えて評価Notebookを作る
https://zenn.dev/microsoft/articles/llm-testing-in-notebook

#### OpenAIのモデルアップデートに備えてPrompt Flowでモデルの評価フローを作る
https://zenn.dev/microsoft/articles/llm-testing-using-prompt-flow

## トークン
#### OpenAI 言語モデルで日本語を扱う際のトークン数推定指標
https://zenn.dev/microsoft/articles/dcf32f3516f013

#### C# で Azure OpenAI Service のトークン数を数えよう
https://zenn.dev/microsoft/articles/azure-openai-count-token

## 監視
#### Azure OpenAIのプロンプトをノーコードで監視したい！
https://zenn.dev/microsoft/articles/azure-openai-nocode-logging

#### Azure OpenAI Serviceの死活監視は可能なのか
https://logico-jp.io/2023/09/27/is-it-possible-to-check-if-azure-openai-service-is-healthy/


# アーキテクチャ
## ネットワーク
#### Azure OpenAI Serviceにプライベートエンドポイントから接続する
これがあるから本家OpenAIではなくAzure OpenAI Serviceを選択する企業も多い、ネットワーク閉域化ができるAzureの「プライベートエンドポイント」。Azure OpenAI Serviceにも対応しています。
https://zenn.dev/microsoft/articles/198989f60eba61

#### Azure OpenAI Serviceを拠点から閉域で使う方法
こちらの記事は上記のプライベートエンドポイントの記事に加えてVPN経由でオンプレミスからも疎通ができるようにする方法も記載されています。DNSをどう構成するか、VNET設計をどうするかというところまで詳しく解説されています。
https://qiita.com/hiro10149084/items/cf18c63dbddb1d7820c7

## API Management
#### Azure OpenAI ServiceからのHTTPステータスに応じて、Azure API Managementで再試行したりフォールバックしたりしたい
タイトルの通りです。API Managementを使って１つのAzure OpenAI Serviceがエラーを返しても別のAzure OpenAI Serviceにフォールバックする仕組みを作る方法が記載されています。
https://logico-jp.io/2023/08/02/use-azure-api-management-policies-to-retry-and-fallback-azure-openai-service-based-on-http-status/

### Azure API Management でAzure OpenAI のChatGPTを取り扱う
API Management内であらかじめOpenAIの認証情報をセットしておいて通信するような構成を作る方法が記載されています。
https://level69.net/archives/33697

#### OpenAI PythonライブラリをAzure API Managementと共に使うメモ
Azure API ManagementをAzure OpenAI Serviceとクライアントの間にデプロイするアーキテクチャはよくあるパターンですが、クライアントのSDKによってはAPI Managementのエンドポイントが/openaiで終わる必要や、認証ヘッダーを変換する必要があります。その場合のAPI Managementの構成方法が記載されています。
https://zenn.dev/microsoft/articles/19de2adf4f36d3

#### Azure OpenAI Serviceをバックエンドサービスとして構成したAzure API ManagementのAPIに対し、Streamを有効にしたリクエストを投げたらHTTP 500を返してしまうことがある
Azure OpenAI ServiceのStreamをtrueにしてリアルタイムでチャットが流れるようにしておくとServer Side Event(SSE)が使用されるが、その場合にAPI Managementで一定の設定(トークン数チェックなど)をしているとHTTP 500が返ってしまう事象（バグではなくドキュメントにも記載がある仕様）についての記事です。
https://logico-jp.io/2023/09/12/when-invoking-apis-hosted-by-azure-api-management-configured-aoai-as-a-backend-service-with-requests-in-which-stream-is-set-to-true-azure-api-management-might-return-http-500/


## 負荷分散
#### Azure OpenAI Serviceへの負荷分散
Azure OpenAI Serviceってクオーター制限が結構厳しいんですよね、ってことで複数のOpenAIリソースの間で負荷分散をしましょう。という記事です。Azure Load BalancerなどAzureのサービスと連携して負荷分散を行う方法が記載されています。
https://logico-jp.io/2023/06/08/request-load-balancing-for-azure-openai-service/

## Retrieval Augmented Generation(RAG)アーキテクチャ
#### Microsoft が Azure Cognitive Search による RAG システムの定量評価結果を公表
https://qiita.com/nohanaga/items/95a4780757e89b29e93e

#### LangChainでCognitive SearchのベクトルDBと連携させたRAGを構築する
https://qiita.com/tmiyata25/items/bfb7f4f5f22ec659c791

#### 新しくなった ChatGPT × Cognitive Search を使ったエンタープライズサーチ
https://qiita.com/nohanaga/items/bbc0c1fba1ec53deeeed

#### Azure Cognitive Search × Azure OpenAI Service エコシステム
https://qiita.com/nohanaga/items/39a1d97cc490b0754df3

# SDK / ライブラリ
## 全般
#### Langchain・Semantic Kernel・guidanceでエージェント機能を実装して比較してみた。
https://qiita.com/sakue_103/items/6ffee0bc267e71eafd60

## OpenAIライブラリ
#### Azure.AI.OpenAI パッケージで OpenAI と Azure OpenAI Service に繋ぐときの違い
https://zenn.dev/microsoft/articles/howtouser-azure-ai-openai

#### Azure OpenAI Service の C# SDK (ChatGPT でも使えます)
https://zenn.dev/microsoft/articles/azure-openai-service-csharpsdk

#### openai の v1 系での Azure OpenAI へのアクセス (Python)
本家OpenAIのPythonライブラリはバージョンが1系にアップデートされたタイミングでいくつかの破壊的変更が入りました。
バージョン1系のライブラリを使ってAzure OpenAI Serviceにアクセスする方法が記載されています。
https://blog.beachside.dev/entry/2023/11/13/130000

#### Azure OpenAI Service の Completions で入力で渡したテキストを結果に含める方法
OpenAIClientのCompletionsでリクエストを行うと通常はGPTからの出力のみしかレスポンスに含まれませんが、入力したプロンプトもレスポンスに含ませる方法が記載されています。
https://zenn.dev/microsoft/articles/azure-openai-do-not-echo

## Semantic Kernel
#### Semantic Kernel が Copilot stack との連携を明確化しプラグインエコシステムと統合
https://qiita.com/nohanaga/items/3c9446375491f46e4aae

#### Azure OpenAI Service を便利に使うための Semantic Kernel を試してみよう on C#
https://zenn.dev/microsoft/articles/semantic-kernel-1

#### Semantic Kernel のスキルを外部ファイルで定義して読み込む
https://zenn.dev/microsoft/articles/semantic-kernel-2

#### Semantic Kernel のネイティブ スキルを試してみよう
https://zenn.dev/microsoft/articles/semantic-kernel-3

#### Semantic Kernel のスキルを好きにカスタマイズしよう (余談です)
https://zenn.dev/microsoft/articles/semantic-kernel-4

#### Semantic Kernel でトークンを数える
https://zenn.dev/microsoft/articles/semantic-kernel-5

#### Semantic Kernel でプランナーを使って自発的に解決策を考えて実行する AI を作ってみよう
https://zenn.dev/microsoft/articles/semantic-kernel-6

#### Semantic Kernel のテンプレートをプログラムでレンダリングする方法とテンプレートの文法
https://zenn.dev/microsoft/articles/semantic-kernel-7

#### Semantic Kernel で Open AI の Embeddings を使う (あいまい検索出来てすげーやつ)
https://zenn.dev/microsoft/articles/semantic-kernel-8

#### Semantic Kernel で自作スキルをクラスライブラリ形式で共有したい
https://zenn.dev/microsoft/articles/semantic-kernel-9

#### Semantic Kernel でトークンの限界を超えるような長い文章を分割してスキルに渡して結果を結合したい
https://zenn.dev/microsoft/articles/semantic-kernel-10

#### Semantic Kernel の WithMemoryStorage と WithMemory の違い
https://zenn.dev/microsoft/articles/semantic-kernel-11

#### Semantic Kernel で Bing Chat みたいに自然言語から検索をして結果を自然言語で返したい
https://zenn.dev/microsoft/articles/semantic-kernel-13

#### Semantic Kernel の Plan ディープダイブ
https://zenn.dev/microsoft/articles/semantic-kernel-14

#### Semantic Kernel で Plan を進捗情報も含めて永続化したい
https://zenn.dev/microsoft/articles/semantic-kernel-15

#### Semantic Kernel の 0.17.230626.1-preview のネイティブ関数の引数に関する新機能
https://zenn.dev/microsoft/articles/semantic-kernel-16

#### Semantic Kernel を使ってアプリ内に AI を組み込んでみた
https://zenn.dev/microsoft/articles/semantic-kernel-17

#### Semantic Kernel の Function calling サポートを試してみよう
https://zenn.dev/microsoft/articles/semantic-kernel-18

## LangChain
#### LangChain の Vectorstore として Azure Cache for Redis を使ってベクトルの格納と検索を行う
https://zenn.dev/microsoft/articles/6d4a04a6f45d4d

#### LangChainとGPTで質疑応答を楽にしたい！
https://qiita.com/tmiyata25/items/8c6d2769ccfb208989d2

#### LangChainとAzure OpenAI版GPTモデルの連携部分を実装してみる
https://qiita.com/tmiyata25/items/7a04096342241d8a2b4c


# 特定機能
## Vision
#### Azure OpenAI Service で GPT-4 Turbo with Vision を使う
https://zenn.dev/microsoft/articles/cd3060cbcf0303

## Assitant API
https://zenn.dev/microsoft/articles/533488db14e442

## Add your data（On your data）
#### 独自ナレッジをノーコードでChatGPTに連携！Azure OpenAI「Add your data」
https://zenn.dev/microsoft/articles/azure-openai-add-your-data

#### Azure OpenAI on your data でノーコードで ChatGPT 対応エンタープライズサーチを構築する
https://qiita.com/nohanaga/items/a3e236fdbb3ef5878012

#### Azure OpenAI Service On Your Data の仕組みと使う上で気を付けるべきポイント
https://zenn.dev/microsoft/articles/ad14d45121abe7

#### Azure OpenAIの「Add your data」で出来ること出来ないこと
https://zenn.dev/microsoft/articles/azure-openai-add-your-data-procon

#### Azure OpenAIの「Add your data」をAPIとして使う！
https://zenn.dev/microsoft/articles/azure-openai-add-your-data-api

#### Azure OpenAI on your dataのAPIでハマった点
https://qiita.com/yahayuta/items/fc51b6a5abb214b68c5a

#### Azure OpenAI Service On your data でベクトル検索を行う
https://zenn.dev/microsoft/articles/06063068b75a91

## Function calling
#### Azure OpenAIでFunction Callingを使う！
https://zenn.dev/microsoft/articles/azure-openai-add-function-calling

### Azure OpenAIのAPIにtoolsプロパティが追加されたぞ！
https://zenn.dev/microsoft/articles/azure-openai-tools

#### Azure OpenAI の Function calling を Golang で試す
https://zenn.dev/microsoft/articles/aoai-function-calling-with-golang

####  Azure OpenAI に Function calling が来たので .NET SDK で動作確認してみた
https://zenn.dev/microsoft/articles/dotnet-sdk-openai-functioncalling

#### 自作したChatGPT規格のプラグインをFunction callingを使って呼び出す方法
https://qiita.com/sakue_103/items/8f3818779c6eb1bb178a

#### OpenAI に毎回返事をしてほしくないという要望を実現してみよう
https://zenn.dev/microsoft/articles/openai-reply-only-if-i-requested

## コンテンツフィルター
#### Azure OpenAI Service の設定可能なコンテンツフィルター
https://zenn.dev/microsoft/articles/ad8fc1c403ced1


# Power Platform連携
## 全般
#### Power Platform × Azure OpenAI の価値
https://qiita.com/Takashi_Masumori/items/bef0a07a521cae238064

#### Power Platform から VNet を使用して Azure OpenAI へ通信する
https://qiita.com/Takashi_Masumori/items/a2c716c8529aca9993ef

#### Power Platform から Azure OpenAIの新機能「Add your data」を利用する
https://qiita.com/Takashi_Masumori/items/410e7e92e6f93ec823c2

#### Power Platform から Azure OpenAI の Function Calling を利用してみよう
https://qiita.com/Takashi_Masumori/items/3ac1b28d4de539f9952a

## Power Apps
#### Power Apps で ChatGPT と連携する実用的なアプリを作ってみました
https://qiita.com/Takashi_Masumori/items/8a07fb6d1869967ce1f9

#### Azure OpenAI Service の SDK を利用し、フロントエンドを Power Apps にする
https://qiita.com/Takashi_Masumori/items/12095390a633b8789522

#### Power Apps で ChatGPT のマークダウンの回答を上手く表示する
https://qiita.com/Takashi_Masumori/items/e9739e545bfc0c7c2c81

#### Power Apps で AI Builder 経由で ChatGPT 連携する (会話履歴と文脈に基づいた回答を生成する)
https://qiita.com/Takashi_Masumori/items/173cb84a860b491f885b

## Power Automate
#### Power Platform から Azure OpenAI Service 経由で ChatGPT を利用してみる
https://qiita.com/Takashi_Masumori/items/96b9e0f1590762fd6fad

### Power Automate から Azure OpenAI にリクエストをする際に文字列を分割する
Azure OpenAI Serviceの各モデルには最大トークン数が決められており、入力出力できる文字数に上限があります。
それを超えた文量を扱いたい場合、再起的要約と呼ばれる手法があります。
再起的要約は入力文を分割してそれぞれモデルで要約を行い、出力文を結合してさらに要約することで上限を超えた文量を扱うことができます。
https://qiita.com/Takashi_Masumori/items/d05e8ae5a4056e2f81de

## Power Virtual Agents
#### Power Virtual Agents を使用して Azure OpenAI Service の ChatGPT と連携し、会話履歴と文脈に基づいた回答を生成する方法
https://qiita.com/Takashi_Masumori/items/975ed38be5879e2ee5c8

#### PVA と Azure OpenAI の ChatGPT：会話履歴の管理術
https://qiita.com/Takashi_Masumori/items/f7b515da9e37a3c1336a

#### Azure OpenAI の 「Add your data」と Power Virtual Agents の 「Boost conversations」を比べてみた
https://qiita.com/Takashi_Masumori/items/5405150b89335008669a


# ハンズオン資料/サンプル
## ハンズオン資料
#### まだOpenAI使ったことないの？この記事で全員ハンズオンさせてやんよ！
Azure DevOpsを使用してAzure Static Web AppsにアプリをデプロイしてAzure OpenAI Serviceを使うハンズオンです。Azure DevOpsもAzure Static Web Appsも無料枠があるので、Azure OpenAI Service分の課金だけで試すこともできそうです。
https://qiita.com/fe_js_engineer/items/4c11827906d38051ae51

#### デジタル庁　ChatGPTを業務に組み込むためのハンズオン
すでに環境が準備された状態での主にプロンプトエンジニアリングをテーマとしたエンドユーザー目線のハンズオンです。補足としてRAGアーキテクチャについても少しだけ触れられています。
https://www.digital.go.jp/assets/contents/node/information/field_ref_resources/5896883b-cc5a-4c5a-b610-eb32b0f4c175/82ccd074/20230725_resources_ai_outline.pdf

#### Azure OpenAI Service を利用した ChatGPT お試し環境の構築
とりあえず閉域化したChatGPTを作りたい！という方はこちら。
https://ks6088ts.github.io/blog/fork-azure-openai-playground

#### Azure OpenAI Service を利用した ChatGPT お試し環境にログ出力機能を追加する
上記の閉域化ChatGPTにAPI Managementをデプロイしてログ出力機能を追加するハンズオンです。
https://ks6088ts.github.io/blog/aoai-apim-easyauth

#### Azure OpenAI Service を用いた Function Calling のハンズオン
pythonを使用してAzure OpenAI ServiceのFunction Callingの基礎を体験できるハンズオンです。
https://ks6088ts.github.io/blog/aoai-function-calling

#### Azure OpenAI ✖️ LINE MessagingAPI を使って自分だけのAIチャットボットを作ろう！
https://zenn.dev/yusu29/articles/azure_openai_chatbot_handson

## サンプル
#### Azure OpenAIのサンプルまとめ
主にGitHubで公開されているデプロイ・実行可能なサンプルソリューションがまとめられています。
https://zenn.dev/microsoft/articles/5c937466f09d98

#### Azure OpenAI Samples Japan
公式情報にはなりますが、[Azure OpenAI Serviceリファレンスアーキテクチャ](https://www.microsoft.com/ja-jp/events/azurebase/blog/aoai-referencearchitecture-release/)の実装サンプルリポジトリです。
https://github.com/Azure-Samples/jp-azureopenai-samples

#### azureopenai-essential-notebooks
筆者がオーナーになっているAzure OpenAIを使ったさまざまなシナリオの基本を学ぶためのプリミティブなJupyter Notebookサンプル集です。
https://github.com/07JP27/azureopenai-essential-notebooks

#### ChatGPT-Prompt-Sample-Japanese
Azure OpenAIには限りませんが、ChatGPTで使用できるシナリオ別プロンプトサンプル集です。
https://github.com/dahatake/ChatGPT-Prompt-Sample-Japanese

#### 社内向けChatGPTを費用を押さえつつ爆速で構築する方法をMicrosoftが提供してくれていた件
Microsoftの公式リポジトリで公開されているChatGPTサンプルアプリをデプロイして使う方法が記載されています。
https://qiita.com/keitomatsuri/items/aeacb527fbc5de34cacc

#### Azure-Samples/azure-search-openai-demo でログを有効にする
公式サンプルのAzure-Samples/azure-search-openai-demoリポジトリにログ出力機能を追加する方法が記載されています。
https://qiita.com/georgeOsdDev@github/items/c8a67c9bd897c6b5b003

#### Azure OpenAI を使った知識埋め込み型QAボットのサンプルをいじってみた
https://qiita.com/natsu_san/items/900473c98049985fa369

#### 【Azure OpenAI開発】 Azureサービスを組み合わせてPDFと対話する簡単なアプリケーションを構築する
https://qiita.com/sakue_103/items/734db0fe72fbb2985ecb

#### 【Azure OpenAI開発】Static Web Apps × Azure Functionsを使って、WebアプリケーションにAzure OpenAIを組み込む
https://qiita.com/sakue_103/items/2b4eeb7fa6384f68e132

#### Redmine で課題が起票された際に Azure OpenAI Service を利用して自動返信する
https://zenn.dev/skmkzyk/articles/openai-redmine-plugin

#### 超長文をChatGPTに要約させてみる
https://qiita.com/shyamagu/items/67ef879c313bcccc9109

#### ChatGPT にプログラムを分析・可視化させてみた
https://qiita.com/shyamagu/items/0f9fea01e5f9aeb685fd

#### 【ChatGPT縛りプレイ】システム開発をさせてみた
https://qiita.com/shyamagu/items/77d527ab43fb57d48e42

#### OpenAIとMicrosoft Graph Search APIでM365の組織内データを検索するRAGアプリを作る
https://zenn.dev/microsoft/articles/azure-openai-graph-rag-pattern

# 事例
#### 【事例大全】Azure OpenAI を利用する Digital Native/Startup まとめ
https://qiita.com/DDKK/items/e47a5db850551844cb82

# 番外編
## Copilot
#### Microsoft の Copilot stack とは何か
https://zenn.dev/microsoft/articles/8974330fb534ef

#### AI Copilot の時代が到来 - Copilot stack による LLM 中心アプリ開発
https://qiita.com/nohanaga/items/c3e92c9adeca7c84a8d5

#### Azure Cognitive Search を ChatGPT Plugin 経由で呼び出し、AI オーケストレーターと統合する
https://qiita.com/nohanaga/items/afcff2bf4ab95089267d

## Open Interpreter
Open Interpreterをは簡単にいうと本家OpenAIのCode Interpreterと同じようなことをオープンソースツールのOpen Interpreterを使ってローカルPC+各種LLMモデルでもできるようにしようというものです。LLMモデルはAzure OpenAI Serviceのモデルにも対応しています。
#### Azure OpenAI Service + Open Interpreterやってみたレポート
https://zenn.dev/happy_elements/articles/745e339b5f9d99

#### ついに来たローカルAIデータ分析！ Open InterpreterとAzure OpenAI Serviceで機密データも安全にハンドリング
https://qiita.com/hiromitsu_iwasaki/items/ef0f302d87e240d9f880

## その他
#### ChatGPT のプラグイン "Code Interpreter" でセキュリティインシデントを分析する
https://zenn.dev/microsoft/articles/49f83651e84e95

#### Code InterpreterでMicrosoft Sentinelのインシデント一覧を分析するサンプルシナリオです。
https://zenn.dev/microsoft/articles/49f83651e84e95

#### Pandas AIをAzure OpenAI Serviceで動かす
https://zenn.dev/microsoft/articles/1bc40c69634417