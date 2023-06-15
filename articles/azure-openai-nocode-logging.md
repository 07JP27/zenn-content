---
title: "Azure Open AIのプロンプトをノーコードで監視したい！"
emoji: "👀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai","aoai", "logging"]
published: true
publication_name: "microsoft"
---

# はじめに
企業(エンタープライズ)でOpen AIを使用する場合、コンプライアンスやセキュリティの観点から、Open AI{へ|から}のプロンプトを監視したいという要望が出てくる場合があります。
Open AIが提供するのはWeb APIであるため、一般的にはフロントエンドやアプリが必要になり、そのアプリ内でロギングの実装をすることが多いかと思います。
しかし複数のアプリでOpen AIを使用する場合に、それぞれのアプリでロギングの実装をするのは大変ですし、Power Platformなどの{ロー|ノー}コードでOpen AIを使用する場合は実装が難しいなど、ログをアプリ外で取得したいケースがあります。
そこで今回はアプリとOpen AIの間でプロンプトのロギングを行う仕組みをやってみます。
今回はAzure Open AIを使用していますが、本家Open AIでも可能です。


# 余談
Azure Open AI自体にもログを収集する機能がありますが、プロンプトの内容を記録することはできません。
![](/images/azure-openai-nocode-logging/compare.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/architecture/example-scenario/ai/log-monitor-azure-openai#alternatives)

# 構成
ポイントとなるのはAzure API Managementです。このAPI ManagementでAPI Managementを通る通信のログを取ることが可能です。これを利用してOpen AIのプロンプトを監視使用というわけです。

![](/images/azure-openai-nocode-logging/architecture.png)


# 設定
すでにAPI ManagementとAzure Open AI、Log Analyticsワークスペースがデプロイ済みであることを前提とします。

## APIの作成
Azure Open AIのAPIをAPI Manamenetに登録します。ここは今回の本質的な部分ではないので詳しくは書きませんが以下の先人の知恵を参考にすると特に問題なく登録できるのではないかと思います。
https://level69.net/archives/33697

![](/images/azure-openai-nocode-logging/1.png)

## ログ設定
API Managementの[診断設定]→[診断設定の追加]　から診断設定を追加します。
ここでのポイントはログの種類として最低限「Logs related to ApiManagement Gateway」を選択することです。
このログカテゴリがAPI Managementを通る通信のログになります。
そして宛先に用意したLog Analyticsワークスペースを選択します。
![](/images/azure-openai-nocode-logging/2.png)

そしてここが最大のポイントです。
実はこのままだとプロンプト（Request Body/Response Body）の内容がログに出力されません。
プロンプトをロギングするために[API]→[All APIs]→[Setting]→[Azure Monitor]から[Number of payload bytes to log]を8192までの自由な値に設定します。プロンプトのサイズがこの値を超えると切り捨てられます。
最後にSaveをクリックして設定を保存します。

![](/images/azure-openai-nocode-logging/3.png)


# 確認してみる
適当にリクエストを投げたあとにLog Analyticsワークスペースに移動して[ログ]を確認してみます。

とりあえず日時とプロンプトの内容だけ見れればいいのでクエリは以下で。
```
ApiManagementGatewayLogs
| project TimeGenerated, RequestBody
```
無事にプロンプトの内容が出力されていることが確認できました。
![](/images/azure-openai-nocode-logging/4.png)

# 注意点
最大のポイントとして挙げた[Number of payload bytes to log]は既定で0になっていると思います。
これはペイロードの中身をログに出力しないことでLog Analyticsのコストを抑えるための設定です。

Log Analyticsは東日本リージョン・Basic SKUの場合、1GBあたり¥101.859/日（月額￥3,055.77）という価格設定がされています。
よく「とりあえずログ取っておけ！」でコストが爆発する例を見かけますので、ご注意ください。

# まとめ
ここまで読んでいただければ、単純にリクエスト/レスポンスのBodyをログに出力しているだけというのがお分かりいただけたかと思います。
Open AI以外でもAPI Managementのログを使用するとバックエンドとアプリ間の通信の内容を監視することができるので、ぜひ活用してみてください。

# 参考
https://learn.microsoft.com/ja-jp/azure/architecture/example-scenario/ai/log-monitor-azure-openai