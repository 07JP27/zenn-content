---
title: "独自ナレッジをノーコードでChatGPTに連携！Azure Open AI「Add your data」"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'openai', 'chatbot',]
published: true
publication_name: "microsoft"
---

先月のMicrosoft Buildで発表されて注目を集めていた、Azure Open AIでコーディングなしで社内のナレッジを組み込んだ回答をするChatGPTが作成できる機能「Add your data」がパブリックプレビューになりました🎉🎉🎉🎉
https://techcommunity.microsoft.com/t5/ai-cognitive-services-blog/introducing-azure-openai-service-on-your-data-in-public-preview/ba-p/3847000


今まではこのようなアーキテクチャを独自で構築する必要があったため、なかなかハードルが高いものでした。
![](/images/azure-openai-add-your-data/appcomponents.png)

こちらのアーキテクチャはサンプルとして公開されているのでご興味があれば見てみてください。
https://github.com/Azure-Samples/azure-search-openai-demo


# やってみる
ではセットアップしてみましょう。
今回はすでにCognitive Searchのリソースがある前提で進めます。

Azure Open AIのプレイグランドから「チャット」にアクセスして「Add your data」タブから「Add a data source」を選択します。
![](/images/azure-openai-add-your-data/1.png)


構成ウィザードが表示されるのであとはそれに従って作成したCognitive Searchのリソースを選択していくだけです！
![](/images/azure-openai-add-your-data/2.png)

Azure Blobにパワポやwordファイルをアップロードして、そのBlobを選択することで文字読み取り、チャンク分割全て全自動でAzure Cognitive SearchのIndexを作ってくれる機能もあるみたいです（私はまだ未検証ですが）。
https://learn.microsoft.com/en-us/azure/cognitive-services/openai/concepts/use-your-data#data-formats-and-file-types


しばらくすると構成が完了し、もうこの時点でCogntive Searchのデータも使ったチャットが可能になります！(サンプルで物件情報を読み込ませています)
![](/images/azure-openai-add-your-data/3.jpeg)

このままではプレイグラウンドのみでしか使えないので、実際にWebアプリとして公開します。「Deploy to」を選択
![](/images/azure-openai-add-your-data/4.jpeg)

WebアプリをホストするWeb Appsを新規で作るのか、既存のリソースを使うのかを選択してWeb Appsにアプリをデプロイ(公開)します。
公開と言っても既定でAzure AD認証が有効になった状態で公開されるので、誰でもアクセスできる、というわけではありません。
![](/images/azure-openai-add-your-data/5.png)

実際にアプリが使えるようになるまで十数分かかりますが、特に難しいコーディングなくアプリもデプロイできました！Bing Chatのようにreferenceをクリックすると回答に使用したCognitive Searchのデータが表示されます。
![](/images/azure-openai-add-your-data/6.jpeg)


# もう少し詳しくみてみる
デプロイされたWeb Appsをもう少し見てみます。

## 言語はPython & Flask
自動デプロイされたWeb AppsはAzureポータルで確認できます。言語はPythonのようです。
![](/images/azure-openai-add-your-data/7.png)

自動デプロイされたWebアプリのソースコードをKuduで見てみるとフレームワークにFlaskが使われているのがわかります。
![](/images/azure-openai-add-your-data/8.jpeg)

## Web APIも提供
冒頭に掲載したプレビューアナウンスには以下のように書かれていました。
> You can use the API and endpoint to integrate this feature in the service, not just in an app.(APIとエンドポイントを利用することで、アプリだけでなく、サービス内でこの機能を統合することができます。)

ということで改めてコードを見てみると、自動デプロイされたWebアプリの`/conversation`にリクエストすることで**独自ナレッジ反映済みのChatGPT APIとしても利用できる**ようです。（既定だとAAD認証が付いているのでAPIを呼ぶ際も認証が必要です。）
![](/images/azure-openai-add-your-data/9.png)

こちらのポイントについては別途記事にしています。
https://zenn.dev/microsoft/articles/azure-openai-add-your-data-api

## プロンプトが・・・ない！？（この記事のメイントピック）
独自ナレッジを組み込んで回答する場合、「以下の情報からのみ回答を考えて〜」みたいな質問応答プロンプトをつけるのが一般的です。
このアプリはどんなプロンプト投げてるのか気になったので見てみたのですが、アプリの中にプロンプトはありませんでした。
ではどうしているのかというと、APIとして隠蔽されています(!)

Azure Open AIのRESTリファレンスをみると`dataSources`なるプロパティがさらっと追加されています。
ここに対して独自データのエンドポイント情報をセットして、ユーザーの入力とと共にリクエストすると、あとはAzure Open AIが勝手にやってくれる、という仕組みです。（執筆時点では日本語ドキュメントには未反映）
https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#completions-extensions
![](/images/azure-openai-add-your-data/10.png)

実際にWebアプリのソースコードからもそのプロパティを使用しているのがわかります。
![](/images/azure-openai-add-your-data/12.png)

## システムプロンプトは環境変数で設定できる
Web Appsの「構成」をみると、システムプロンプトは環境変数で設定できるようになっています。
ここを変えることによってキャラクター定義をはじめとしたメタプロンプトが可能です。
![](/images/azure-openai-add-your-data/14.png)

## デプロイされるアプリはGitHubで公開されている
Web Appsの「デプロイセンター」をみると自動デプロイされたアプリのソースコードはGitHubから引っ張ってきていることがわかります。リポジトリにアクセスしてみると、Publicで公開されたMIT licenseのプロジェクトなので、これをForkして自分のアプリを作ることもできます（Pythonですが）。
![](/images/azure-openai-add-your-data/13.png)

リポジトリはこちらです。
https://github.com/microsoft/sample-app-aoai-chatGPT
