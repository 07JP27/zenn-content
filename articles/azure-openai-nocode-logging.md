---
title: "Azure Open AIのプロンプトをノーコードで監視したい！"
emoji: "👀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai","aoai", "logging"]
published: true
publication_name: "microsoft"
---

# はじめに
企業(エンタープライズ)でOpen AIを使用する場合、コンプライアンスやセキュリティの観点からOpen AI{へ|から}のプロンプトを監視したいという要望が出てくる場合があります。
Open AIが提供するのはWeb APIであるため、一般的にはフロントエンドやアプリが必要になり、そのアプリ内でロギングの実装をすることが多いかと思います。
しかし、複数のアプリでOpen AIを使用する場合にそれぞれのアプリでロギングの実装をする方法は冗長かつ一貫性を保つことが難しいですし、Power Platformなどの{ロー|ノー}コードでOpen AIを使用する場合はそもそもロギングの実装が難しい、すでに運用されているアプリを改変することなく監視機能を付け加えたいなど、アプリ外でログを取得したいケースがあります。
そこで今回はアプリなどのコーディングを必要とせず、アプリとOpen AIの間でプロンプトのロギングを行う仕組みをやってみます。
今回はAzure Open AIを使用していますが、本家Open AIでも可能です。


# 余談
Azure Open AI自体にもログを収集する機能がありますが、プロンプトの内容を記録することはできません。
![](/images/azure-openai-nocode-logging/compare.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/architecture/example-scenario/ai/log-monitor-azure-openai#alternatives)

# 構成
ポイントとなるのは[Azure API Management](https://learn.microsoft.com/ja-jp/azure/api-management/api-management-key-concepts)です。API Managementは名前の通りAPIを管理するためにサービスで、複数のバックエンドを１つのエンドポイントのまとめたり、APIに独自のスロットリングをかけたりなどAPIエコシステムを実現できます。
API Managementでは監視の機能としてAPI Managementを介して通信されるトラフィックのログを取ることが可能です。今回はこれを利用してOpen AIのプロンプトを監視しようというわけです。

![](/images/azure-openai-nocode-logging/architecture.png)


# やってみる
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

この問題に対する一般的な対応策として、長期に保管するログは比較的コストが安いストレージアカウントに退避させておき、Log Analyticsでは直近のデータのみ保持しておくというような方法があります。
https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/logs-data-export?tabs=portal#export-destinations

また、そもそもLog Analyticsのようにインタラクティブにクエリをかける必要がない場合は[`診断設定`]の[`宛先`]をストレージアカウントに設定し、最初からストレージに出力してしまうのも手です。
https://learn.microsoft.com/ja-jp/azure/azure-monitor/essentials/diagnostic-settings?tabs=portal#destinations

# まとめ
ここまで読んでいただければ、単純にリクエスト/レスポンスのBodyをログに出力しているだけというのがお分かりいただけたかと思います。
Open AI以外でもAPI Managementのログを使用するとバックエンドとアプリ間の通信の内容を監視することができるので、ぜひ活用してみてください。

# [番外編]ユーザー識別子も記録したい！
Azure ADのJWTトークンなどをつけてリクエストすればリクエストヘッダーのJWTをデコードすることでユーザー識別子の確認が可能です。
でもどうせならデコードした状態でログを記録してしまいましょう。

API Managementのポリシーは以下のような設定になります。
```xml
<policies>
    <inbound>
        <base />
        <validate-azure-ad-token tenant-id="{JWTを取得するためのアプリ登録が存在するテナントのID}" header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="認証に失敗しました" output-token-variable-name="jwt-variables">
            <client-application-ids>
                <application-id>{JWTを取得するためのアプリ登録のアプリケーションID}</application-id>
            </client-application-ids>
            <required-claims>
                <claim name="unique_name" />
            </required-claims>
        </validate-azure-ad-token>
        <trace source="jwt" severity="information">
            <message>@(((Jwt)context.Variables["jwt-variables"]).Claims.GetValueOrDefault("unique_name"))</message>
        </trace>
    </inbound>
    ...
</policies>
```
このポリシーをセットして実際にリクエストをしてみると、以下のようにトレースとしてユーザー識別子が出力されます。
![](/images/azure-openai-nocode-logging/jwt.png)

# 参考
https://learn.microsoft.com/ja-jp/azure/architecture/example-scenario/ai/log-monitor-azure-openai