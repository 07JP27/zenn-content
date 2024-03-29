---
title: "Azure OpenAIの「Add your data」で出来ること出来ないこと"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai"]
published: true
publication_name: "microsoft"
---

# はじめに
最近リリースされたAzure OpenAIの「Add your data」は大きな注目を集めています。
**「Add your data」はAzureが提供する何かすごい機能というわけではなく、Cognitive Searchを使用した検索とそこから回答を得るためのプロンプトエンジニアリングなどの実装をAzureがマネージドサービスとして提供しているものです。**
そのため「Add your data」のような機能を自前で実装することも可能で、実際に社内ナレッジ検索を自前実装するサンプルが公開されています。
https://github.com/Azure-Samples/azure-search-openai-demo

今回は同じ独自ナレッジの検索機能付きChatGPTという面から「Add your data」と自前実装の違いについて見ていきます。「Add your data」の機能についてまだご存じない方は以下の記事をご一読ください。
https://zenn.dev/microsoft/articles/azure-openai-add-your-data


# 「Add your data」で出来ること出来ないこと
ここから先の情報は本記事の執筆または更新時点の情報です。出来る限り最新情報に追従したいと思いますが、必ずしも最新ではないことにご注意ください。
また、「Add your data」を基準にして比較してしまっているので「Add your data」でできること、というのが少なくなっています。ご了承ください・・・。

||Add your data|自前実装|
|:--|:--|:--|
|[ノーコードでの実現](#ノーコードでの実現)|〇|△|
|[閉域化](#閉域化)|〇|〇|
|[プロンプト改善](#プロンプト改善)|△|〇|
|[セマンティック検索](#セマンティック検索)|△|〇|
|[日本語の精度向上](#日本語の精度向上)|△|〇|
|[Cognitive Search以外のデータソース](#cognitive-search以外のデータソース)|×|〇|
|[ベクトル検索](#ベクトル検索)|〇|〇|


では各項目について見ていく前に、まずは「Add your data」の仕組みについておさらいしましょう。
「Add your data」のためにAzure OpenAIに[Completions extensions API](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#completions-extensions)というAPIが追加されています。ここに対して回答源となるデータソース（執筆時点ではCognitive Searchのみ）とユーザーの入力を渡すことでクエリの生成から回答作成ができます。
![](/images/azure-openai-add-your-data-procon/addyourdata.png)

## ノーコードでの実現
「Add your data」はAzure OpenAI StudioからノーコードでフロントのWebアプリ発行までが可能です。これについては今までも多くの方が記事を執筆しているので改めてここでは説明しませんが、以下の公式クイックスタートをなぞるとコードを一行も書くことなく構築の体験ができます。

https://learn.microsoft.com/ja-jp/azure/cognitive-services/openai/use-your-data-quickstart?tabs=command-line&pivots=programming-language-studio

一方、自前実装の方はサンプルリポジトリを使用することでazdコマンド1発で構築はできるものの、**あくまでサンプル**という立ち位置のため、開発者によるカスタマイズや教材となることを前提としていると考えられます。そのため、ノーコードでの実現という点では「△」としています。
https://github.com/Azure-Samples/azure-search-openai-demo


## 閉域化
Azureには[Private Endpoint](https://learn.microsoft.com/ja-jp/azure/private-link/private-endpoint-overview)や[VNet統合](https://learn.microsoft.com/ja-jp/azure/app-service/overview-vnet-integration)など、PaaSサービスを仮想ネットワーク内に閉じ込め、インターネットからのアクセスを制限(=閉域化)する機能があります。今回の登場人物のWebアプリをホストしているWeb Appsや、Cognitive Search、Azure OpenAIもこれら機能に対応しています。
||Private Endpoint|VNet統合|
|:--|:--|:--|
|Web Apps|〇|〇|
|Cognitive Search|〇|-|
|Azure OpenAI|〇|-|

この記事の主題ではないので簡単に説明しますが、PaaS**への**アクセスを閉域化するのがPrivate Endpoint、PaaS**からの**アクセスを閉域化するのがVNet統合です。

:::details 非対応だった以前までの内容
では、登場人物全てが閉域化に対応しているのに、なぜ「Add your data」は閉域化できないのでしょうか。それは**Azure OpenAIからCognitive Searchへの通信が執筆時点ではパブリックのみになっているから**です。「Add your data」の仕組み図を閉域ネットワーク的に書き換えると以下の図のようになります。
![](/images/azure-openai-add-your-data-procon/privatenwrestrict.png)
ネットワーク閉域化をしている場合、インターネットからのアクセスを遮断するのでAzure OpenAIからのインターネット経由のアクセスができなくなります。そのため、執筆時点では「Add your data」は閉域化できないということになります。Azure OpenAIがVNet統合などに対応してAzure OpenAI**からの**アクセスがVNet経由でできるようになれば、閉域化が可能になるかもしれませんが、執筆時点ではそのロードマップは公開されていません。

なお、図で「基本的には」と記載しているのはPrivate Endpointを構成している場合でもアクセス制限の設定によってパブリックアクセスを許可できる(VNet内からはPrivate Endpoint経由、VNet外からはインターネット経由)からです。しかし、それではエンドポイントを公開することになり、閉域化しているとはいえないので、あくまで閉域化できないということになります。
:::


プライベートネットワークで保護されたAzure OpenAIリソースをお持ちで、そのデータ上でAzure OpenAIによるCognitiveへのアクセスを許可したい場合は、[申請フォーム](https://forms.office.com/pages/responsepage.aspx?id=v4j5cvGGr0GRqy180BHbRw_T3EIZ1KNCuv_1duLJBgpUNFlLNEo2MVhRMzNMTjJHTTlQTEk0QTYzOSQlQCN0PWcu)から申請することで構成が可能です。申請書は5営業日以内に審査され、その結果がメールで送信されます。

仕組みとしては申請が承認されると、申請したリソースに対してMicrosoftからプライベートリンクを貼るためのリクエストが飛んできます。これを承諾することで「Add your data」のトラフィックがCognitive Searchのプライベートエンドポイントへアクセスすることができるようになり、結果的にCognitive Searchへのパブリックアクセスを遮断できます。
https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/use-your-data#virtual-network-support--private-network-support

仕組みのイメージとしては以下のような感じです。（Microsoft側のVNetやPrivate Endpointがブラックボックスなのであくまでイメージです）
![](/images/azure-openai-add-your-data-procon/privateendpoint.png)



自前実装の場合、Cognitive Searchへのアクセスも開発者が制御できるWeb Appsなどのリソースが行うことになるのでVNet統合を利用して閉域化が可能です。
![](/images/azure-openai-add-your-data-procon/privatenwworkaround.png)
上記の図はネットワーク的な図として簡略化されていますが、実際には下図のようにクエリ生成や回答生成のためにAzure OpenAIとの間で複数回やりとりが発生します。
![](/images/azure-openai-add-your-data-procon/addyourdataarch.png)

公式ドキュメントにも[プライベートネットワーク接続はサポートされていない旨](https://learn.microsoft.com/ja-jp/azure/cognitive-services/openai/concepts/use-your-data#virtual-network-support--private-link-support)の記述が追加されました。



## プロンプト改善
「Add your data」の仕組みでは、プロンプトが大きく2箇所で使われます。
- ユーザーの入力からCognitive Searchへ投げるためのクエリを生成するとき
- Cognitive Searchからの回答を元にユーザーに返す回答を生成するとき

そして**いずれのプロンプトも「Add your data」のマネージド機能として隠蔽されています。**

「Add your data」でユーザー入力以外にカスタムできるプロンプトは通常のChatGPTでも使われる "role": "system"のシステムプロンプトとCompletions extensions APIのプロパティとして存在する[roleInformation](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#example-response-3)だけですが、こちらはそれぞれ[200トークン](https://learn.microsoft.com/ja-jp/azure/cognitive-services/openai/concepts/use-your-data#system-message)と[100トークン](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#completions-extensions:~:text=There%E2%80%99s%20a%20100%20token%20limit)の上限があり、あまりハイコンテキストを与えることができません。
ちなみにAzure OpenAI Studioの「Deploy to...」ボタンによって自動デプロイされるアプリはそれぞれに同じ値（環境変数：AZURE_OPENAI_SYSTEM_MESSAGE）を代入しています。
- ["role": "system"部分のコード](https://github.com/microsoft/sample-app-aoai-chatGPT/blob/ab5fab37cafa10b3b5bebde3537ba3f04f543752/app.py#L195)
- [roleInformation部分のコード](https://github.com/microsoft/sample-app-aoai-chatGPT/blob/ab5fab37cafa10b3b5bebde3537ba3f04f543752/app.py#L83)


とはいえ、そんなシステムプロンプトにもベストプラクティスはあって、[公式ドキュメントの推奨設定](https://learn.microsoft.com/ja-jp/azure/cognitive-services/openai/concepts/use-your-data#system-message)や[多言語のサポート](https://learn.microsoft.com/ja-jp/azure/cognitive-services/openai/concepts/use-your-data#interacting-with-the-model)によると以下のようなシステムプロンプトが推奨されているようです。それぞれトークン上限が異なるので別々のプロンプトを入れてみるのもありかもしれません。

> あなたは、情報の検索を支援する AI アシスタントです。 日本語のドキュメントを取得し、それを日本語で注意深く読み、日本語で回答する必要があります。

自前実装の場合は言わずもがなプロンプト含め自前で実装するので、考えることはSemantic KernelやLangChainを使って楽をするかどうかくらいになってきます。

## セマンティック検索
Cogntive Searchにはセマンティック検索という機能があります。これは「セマンティック」の名の通り「意味的に」近しい検索を行うものです。全文検索とは異なり違うワードや文章でも意味的に近しい答えを回答してくれるため、回答精度の向上が期待できます。
https://learn.microsoft.com/ja-jp/azure/search/semantic-search-overview

「Add your data」に使用されているAPIのプロパティにもCognitive Searchのセマンティック検索を有効にするプロパティがありますが、現在は英語のみのサポートとなっています。
![](/images/azure-openai-add-your-data-procon/semanticlimit.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/cognitive-services/openai/concepts/use-your-data#semantic-search)

Cognitive Searchに対して日本語のセマンティック検索を行うこと自体はできるので、自前実装の場合はセマンティック検索が可能という形になります。
ただし、セマンティック検索機能は記事執筆時点でプレビュー機能となっていますのでご注意ください。
https://qiita.com/nohanaga/items/4b9506c62b9e5796f405


## 日本語の精度向上
「Add your data」の場合、以下のような精度を左右するポイントがあります。
- 検索クエリ/プロンプトの質
- 独自ナレッジの質

[プロンプト改善](#プロンプト改善)で前述したようにプロンプトや検索クエリをカスタムすることができません。よって後者の独自ナレッジの質を上げることしかできません。特に日本語であればアナライザーやインデクサーを工夫することで精度向上が期待できます。精度に関わるポイント２つのうち１つのみカスタム可能なので「△」としました。
「Add your data」で使用されるCognitive Searchの日本語精度向上は以下の記事をご覧ください。
https://qiita.com/tmiyata25/items/e8866dfed6dd4b9a02ad


独自実装の場合、精度向上のポイントとなる２つが完全に制御できるのでお好きなようにカスタムできます。


## Cognitive Search以外のデータソース
「Add your data」に限らずLLMに外部データを取り込む「Grounding」と呼ぶ手法が出てきています。例えばMicrofot Graph APIを使用して組織内のユーザーやグループを検索して自然言語で回答する、というようなシナリオです。
執筆時点でCompletions extensions APIの`dataSources`の`type`プロパティには「AzureCognitiveSearch」のみしか指定できません。つまり「Add your data」では**Cognitive Searchのみ**をサポートしているともいえます。これについても今後増えてくることを期待したいと思います。
![](/images/azure-openai-add-your-data-procon/datasource-type.png)

自前実装では生でデータソースのAPIを叩くなり、Semantic KernelのSkillとして定義してPlannerに食わせるなりして、外部データを取り込むことができます。

【余談】 
ちょっとハッキーかつサポート外の方法だとは思いますが、Cogntiive SearchのAPIと同じインターフェースのAPIを独自で用意して、それをラッパーとして中で別の所望のAPIを呼び出すようにして、Completions extensions APIの`dataSources`にそのAPIを指定すれば独自APIを「Add your data」に組み込めるんじゃないかと思っています。これは今度また試してみようと思います。

## ベクトル検索
「Add your data」の発表と同時期にCognitive Searchでベクトル保存・検索ができる機能が発表されました。詳しくは以下の記事をご覧ください。
https://qiita.com/nohanaga/items/f710cac82072b63bc73f

~~しかし、執筆時点で「Add your data」の[Completions extensions API](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#completions-extensions)にベクトル検索を有効化するようなプロパティがありません。よってベクトル検索は現時点でサポートしていないものと考えられます。~~

2023年8月のアップデートで「Add your data」でのベクトル検索のサポートが発表されました。[Completions extensions API](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#completions-extensions)の`dataSources`→`parameters`プロパティに`embeddingEndpoint`プロパティや`embeddingKey`プロパティが追加されています。

実際の使い方などは以下のブログで解説されています。
https://zenn.dev/microsoft/articles/06063068b75a91


自前実装の場合はEmbedding APIを使ってお好きなようにベクトル化やコサイン類似度計算を実装できます。ベクトル検索(Embedding)のサンプルも公開されていますので、気になる方はチェックしてみてください。
https://github.com/Azure-Samples/azure-open-ai-embeddings-qna

# まとめ
「Add your data」はノーコードでの独自ナレッジ検索の実現が可能ですが、その分機能は限定的です。自前実装の場合は、プロンプト改善やセマンティック検索などの機能を追加することが可能ですが、その分実装コストがかかります。要件に合わせて使い分けることでスピーディーかつ堅牢なシステム構築が可能になります。

# 公式教材（無料）
Microsoftが提供しているEラーニングサイト「MS Learn」にて、Azure OpenAIのAdd your dataに関する無料教材が公開されています。
ご自身のAzure環境を持っていなくてもサンドボックス環境でハンズオンを行うことができますので、ぜひご活用ください。
https://learn.microsoft.com/en-us/training/modules/use-own-data-azure-openai/