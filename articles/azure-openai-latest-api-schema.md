---
title: "Azure OpenAIの最新APIスキーマを確認する方法"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['openai', 'azure']
published: true
publication_name: "microsoft"
---

# 何が問題か
Azure OpenAIのAPIバージョンはとても早いスパンで更新されています。
それゆえ、公式ドキュメントへの反映が間に合っていないことがあります。
例えば以下の画像は、2023/12/7時点での公式ドキュメントに記載されている「サポートされているAPIバージョン」の一覧です。
実際にはこの時点で以下の３つのバージョンがドキュメントに掲載されいないが、実際にはすでに定義されてCallできる状態になっています。
- 2023-09-01-preview
- 2023-10-01-preview
- 2023-12-01-preview


![](/images/azure-openai-latest-api-schema/docs.png)


この記事では、最新のAzure OpenAIのAPIバージョンとその中のインターフェース（スキーマ）を確認する方法を紹介します。


# 確認する方法
Azure OpenAIに限らずAzureサービスのAPIインターフェースは基本的にはSwaggerで定義されてGitHubで公開されているのでこれを使います。
https://github.com/Azure/azure-rest-api-specs/tree/main

Azure OpenAIのAPIバージョンには安定版の**stable**とプレビュー版の**preview**があります。
特にPreviewは安定版ではないためリリースされてもアナウンスが大々的に行われることは少ない一方で、最近だとJSON Modeが使えるようになったり、と最新の機能がPreviewでリリースされていることが多いです。ただしpreview版はその名の通りプレビュー状態なので、問題が発生した場合でもサポートが受けられない可能性や、仕様が変更されたり、早めにディスコンになったりする可能性があります。

以下のリンクからStable,PreviewのAPI定義を確認できます。パスを見るとわかりますがどちらも上記のAzureのAPI定義が置かれているリポジトリの中にあります。

#### Stable
https://github.com/Azure/azure-rest-api-specs/tree/main/specification/cognitiveservices/data-plane/AzureOpenAI/inference/stable

#### Preview
https://github.com/Azure/azure-rest-api-specs/tree/main/specification/cognitiveservices/data-plane/AzureOpenAI/inference/preview


上記のいずれかにアクセスすると、APIのバージョン一覧が表示されるので確認したいバージョンを選択します。
![](/images/azure-openai-latest-api-schema/list.png)

各バージョンの中には`inference.json`ファイルがあるので、それを選択します。
![](/images/azure-openai-latest-api-schema/interface.png)

すると、APIのインターフェース定義が表示されます。これがOpenAPI形式で定義されたそのバージョンのAPIインターフェース定義です。
この定義が確認できれば後はいかようにでもツールを使ってクライアントコードを生成したり、ドキュメントを生成したりできますが、今回Swagger UIを使ってインターフェースを確認だけしてみます。
「Raw」を選択します。
![](/images/azure-openai-latest-api-schema/raw.png)

.jsonで終わるjsonファイルのURLを取得します。
![](/images/azure-openai-latest-api-schema/raw_url.png)


[Swagger UI](https://petstore.swagger.io/)にアクセスして、先ほど取得したURLを入力し、「Explore」をクリックします。
![](/images/azure-openai-latest-api-schema/swagger.png)

これで特定バージョンのAPIインターフェースを確認できました。ちなみに、Swagger UIを使うと実際にAPIを呼び出すこともできます。
![](/images/azure-openai-latest-api-schema/swagger_sample.png)

