---
title: "独自ナレッジをノーコードでChatGPTに連携！Azure Open AI「Add your data」"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'openai', 'chatbot',]
published: true
publication_name: "microsoft"
---

「「やってみた」記事はもう十分」という方は[もう少し詳しくみてみる](#もう少し詳しくみてみる)セクションからご覧ください！

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
自動デプロイされたWeb Appsや裏側の処理をもう少し見てみましょう。

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

こちらのポイントについては以下の記事で詳解しています。
https://zenn.dev/microsoft/articles/azure-openai-add-your-data-api

## プロンプトが・・・ない！？（この記事のメイントピック）
独自ナレッジを組み込んで回答する場合、「以下の情報からのみ回答を考えて〜」みたいな質問応答プロンプトをつけるのが一般的です。
このアプリはどんなプロンプト投げてるのか気になったので見てみたのですが、アプリの中にプロンプトはありませんでした。
ではどうしているのかというと、APIとして隠蔽されています(!)

Azure Open AIのRESTリファレンスをみると、`dataSources`なるプロパティをもつCompletions extensionsというAPIがさらっと追加されています。
ここに対して独自データのエンドポイント情報をセットして、ユーザーの入力とと共にリクエストすると、あとはAzure Open AIが勝手にやってくれる、という仕組みです。（執筆時点では日本語ドキュメントには未反映）
https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#completions-extensions
![](/images/azure-openai-add-your-data/10.png)

実際にWebアプリのソースコードからもそのプロパティを使用しているのがわかります。
![](/images/azure-openai-add-your-data/12.png)

Completions extensions APIについては以下の記事で詳解しています。
https://zenn.dev/microsoft/articles/azure-openai-add-your-data-api

## 日本語精度のポイント
Add your dataの精度を左右するポイントは大きく以下の2(+1)点です。
- Cognitive Searchに格納さた独自ナレッジの質
- 検索クエリの質
- (プロンプトの質)

1つ目については@tmiyata25さんの以下の記事が参考になります。
https://qiita.com/tmiyata25/items/e8866dfed6dd4b9a02ad

2つ目についてもう少し詳しく見てみます。
Add your dataの仕組みでは、まず「ユーザーの入力をクエリ化する」という作業を行います。
例えば「豪華な部屋に住みたい」とユーザーが入力した場合「豪華　部屋　物件」などと言ったようなCognitive Searchへ投げるためのクエリを生成する必要があります。イメージ的には以下の図の通りです（内部の処理が公開されていないのであくまで想像/イメージです）。
![](/images/azure-openai-add-your-data/addyourdataarch.png)

どのようなクエリが投げられているかが気になるところですが、Cognitive Searchの診断設定でログを有効にすると確認できます。
チャットに`めっちゃ豪華な部屋に住みたいです！`と入力した場合
![](/images/azure-openai-add-your-data/17.png)
チャットに`タワマンのペントハウスに住みたい`と入力した場合
![](/images/azure-openai-add-your-data/18.png)

上記を見るとわかる通り、必ずしもそのままではないものの、おおよそそのままの文章がクエリとして投げられていることがわかります。
Cognitive Searchに対しては全文検索を行うため文章でのクエリは有効な回答が見つからない可能性が多いです。
そして、このクエリ生成部分は前述したCompletions extensions APIの中に隠ぺいされているため、開発者は手も足も出ない状況です。
今はプレビューなので英語に最適化されているものと想像しますが、今後GAのタイミングで日本語などのクエリ生成の精度も向上することを期待したいと思います。


## システムプロンプトは環境変数で設定できる
Web Appsの「構成」をみると、システムプロンプトは`AZURE_OPENAI_SYSTEM_MESSAGE`環境変数で設定できるようになっています。
ここを変えることによってキャラクター定義をはじめとしたメタプロンプトが可能です。
ただし、この環境変数はシステムプロンプトのほかに`dataSources`プロパティの`roleInformation`にもセットされるようになっています。[APIドキュメント](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#example-response-3:~:text=There%E2%80%99s%20a%20100%20token%20limit%2C%20which%20counts%20towards%20the%20overall%20token%20limit.)を見ると`roleInformation`は最大100トークンという制約があるため、あまり多くの文字はセット出来なさそうです。
その他にもMax tokenなど変数としてAzure Open AIに渡せるパラメーターは大体が環境変数になっています。
![](/images/azure-openai-add-your-data/14.png)

↓システムプロンプトでの参照
![](/images/azure-openai-add-your-data/16.png)
↓roleInformationでの参照
![](/images/azure-openai-add-your-data/15.png)


## デプロイされるアプリはソースコードがGitHubで公開されている
Web Appsの「デプロイセンター」をみると自動デプロイされたアプリのソースコードはGitHubから引っ張ってきていることがわかります。リポジトリにアクセスしてみると、Publicで公開されたMIT licenseのプロジェクトなので、これをForkして自分のアプリを作ることもできます（Pythonですが）。
![](/images/azure-openai-add-your-data/13.png)

リポジトリはこちらです。
https://github.com/microsoft/sample-app-aoai-chatGPT

## 閉域化の対応
少し上級者向けですが、Cognitive SearchやAzure Open AIではPrivate Linkという仕組みを使ってネットワーク的に閉域化、つまり特定のネットワーク（社内ネットワークなど）からのみアクセスを許可することができます。
特にAdd your dataのような社内ナレッジを含むシステムを構築する場合、閉域化を望むケースも多いと思います。
しかし、Completions extensions APIの場合、APIに対してデータソースとなるCognitive Searchを指定して、Azureのどこかに置かれたCompletions extensions APIの中からCognitive Searchにアクセスできる必要があります。そのため、少なくともこの記事の執筆時点ではAdd your data機能は閉域化に対応していないようです。

記事の冒頭で紹介したサンプルの場合Completions extensions APIの内部で行っている処理を自前で実装する形になるので、その場合は閉域化が可能です。
https://github.com/Azure-Samples/azure-search-openai-demo
