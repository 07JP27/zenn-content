---
title: "独自ナレッジをノーコードでChatGPTに連携！Azure Open AI「Add your data」"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'openai', 'chatbot',]
published: true
publication_name: "microsoft"
---

先月のMicrosoft Buildで発表されて注目を集めていたAzure Open AIでコーディングなしで社内のナレッジを組み込んで解凍するChatGPTを作成できる機能「Add your data」がパブリックプレビューになりました🎉🎉🎉🎉
https://techcommunity.microsoft.com/t5/ai-cognitive-services-blog/introducing-azure-openai-service-on-your-data-in-public-preview/ba-p/3847000


今まではこのようなアーキテクチャを独自で構築する必要があったため、なかなかハードルが高いものでした。
![](https://github.com/Azure-Samples/azure-search-openai-demo/blob/main/docs/appcomponents.png)

こちらのアーキテクチャはサンプルとして公開されているのでご興味があれば見てみてください。
https://github.com/Azure-Samples/azure-search-openai-demo


# やってみる
ではセットアップしてみましょう。
今回はすでにCognitive Searchのリソースがある前提で進めます。

Azure Open AIのプレイグランドから「チャット」にアクセスして「Add your data」タブから「Add a data source」を選択します。
![](/images/azure-openai-add-your-data/1.png)


構成ウィザードが表示されるのであとはそれに従って作成したCognitive Searchのリソースを選択していくだけです！
![](/images/azure-openai-add-your-data/2.png)


しばらくすると構成が完了し、もうこの時点でCogntive Searchのデータも使ったチャットが可能になります！(サンプルで物件情報読み込ませています)
![](/images/azure-openai-add-your-data/3.jpeg)

このままではプレイグラウンドのみでしか使えないので、実際にWebアプリとして公開します。公開と言っても規定でAzure AD認証が有効になった状態で公開されるので、誰でもアクセスできる、というわけではありません。「Deploy to」を選択
![](/images/azure-openai-add-your-data/4.jpeg)

WebアプリをホストするWeb Appsを新規で作るのか既存の者を使うのかを選択してWeb Appsにアプリをデプロイします。
![](/images/azure-openai-add-your-data/5.png)

実際にアプリが使えるようになるまで十数分かかりますが、特に難しいコーディングなくアプリもデプロイできました！Bing Chatのようにreferenceをクリックすると回答に使用したCognitive Searchのデータが表示されます。
![](/images/azure-openai-add-your-data/6.jpeg)


# もう少し詳しくみてみる
デプロイされたWeb Appsをもう少し見てみます。

## 言語はPython & Flask
自動デプロイされたWeb AppsはAzureポータルで確認できます。言語はPythonのようです。
![](/images/azure-openai-add-your-data/7.png)

Kuduで自動デプロイされたWebアプリのソースコードを見てみるとフレームワークにFlaskが使われているのがわかります。
![](/images/azure-openai-add-your-data/8.jpeg)

## Web APIも提供
冒頭に掲載したプレビューアナウンスには以下のように書かれていました。
> You can use the API and endpoint to integrate this feature in the service, not just in an app.(APIとエンドポイントを利用することで、アプリだけでなく、サービス内でこの機能を統合することができます。)

ということで改めてコードを見てみると、自動デプロイされたWebアプリの`/conversation`にリクエストすることでAPIとしても利用できるようです。（規定だとAAD認証が付いているのでAPIを呼ぶ際も認証が必要です。）
![](/images/azure-openai-add-your-data/9.png)


## プロンプトが・・・ない！？（この記事のメイントピック）
独自ナレッジを組み込んで解凍する場合、「以下の情報からのみ回答を考えて〜」みたいな質問応答プロンプトをつけるのが普通です。
このアプリはどんなプロンプト投げてるのか気になったので見てみたのですが、アプリの中にプロンプトはありませんでした。
じゃどうしているのかというと、APIとして隠蔽されています(!)

Azure Open AIのRESTリファレンスをみると`dataSources`なるプロパティがさらっと追加されていて、ここに対して独自データのエンドポイント情報をセットして、ユーザーの入力とと共にリクエストすると、あとはAzure Open AIが勝手にやってくれる、という仕組みです。（執筆時点では日本語ドキュメントには未反映）
https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#completions-extensions
![](/images/azure-openai-add-your-data/10.png)

実際にWebアプリのソースコードからもそのプロパティを使用しているのがわかります。
![](/images/azure-openai-add-your-data/12.png)

## セマンティック検索も対応しそう
同じくMicrosoft BuildでアナウンスされたAzure Cognitive Searchでもベクター検索(Embedding：埋め込み)ができるようになるセマンティック検索機能ですが、今回のWebアプリでも考慮されているようです。環境変数で設定することができるみたいです。
![](/images/azure-openai-add-your-data/11.jpeg)
