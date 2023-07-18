---
title: "Azure Open AIの「Add your data」をAPIとして使う！"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'openai', 'api']
published: true
publication_name: "microsoft"
---

前回の記事で「Azure Open AIの「Add your data」を使用した際に、APIとしても使うことができる」と書きました。この部分についてもう少し詳しくみていきたいと思います。（前回の記事の内容を前提に記載していますのでまだの方は一度お目通しください。）
↓前回の記事
https://zenn.dev/microsoft/articles/azure-openai-add-your-data

# APIとして利用するパターン2つ
「Add your data」をAPIとして使う方法は大きく分けて2つあります。
大きな違いは前回の記事で説明したdataSourcesについて、「**dataSourcesプロパティをどこで指定するか**」という点です。

## Completions extensions APIを直接叩くパターン
Azure Open AIのAPIとして[Completions extensions](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#completions-extensions)というインターフェースが追加されました。
これを使用することでクライアントからリクエストするたびにデータソースを指定することができます。
![](/images/azure-openai-add-your-data-api/2.png)


## 「Add your data」から自動デプロイしたWeb Appsに対して、APIリクエストをするパターン
[前回の記事で確認した](https://zenn.dev/microsoft/articles/azure-openai-add-your-data#web-api%E3%82%82%E6%8F%90%E4%BE%9B)通り、Azure Open AIプレイグラウンドから自動デプロイしたWeb Appsには/conversationエンドポイントが生えているので、これを叩きます。
すでにどのCognitive Searchのデータソースを使うかはWeb Appsの中で指定済みなので、通常のChatGPTのような形でリクエストを投げると、独自ナレッジも含んだ回答が返ってきます。つまり**Web Appsでラップされている状態**ということですね。
![](/images/azure-openai-add-your-data-api/1.png)


# やってみる
## Completions extensions APIを直接叩くパターン
このパターンは[ドキュメント](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#completions-extensions)にも記載されている素のAPIを叩くだけなので特に説明は不要かと思います。
以下のような形で叩くことができます。レスポンス形式もドキュメントに記載されている通りです。
```
curl -i -X POST YOUR_RESOURCE_NAME/openai/deployments/YOUR_DEPLOYMENT_NAME/extensions/chat/completions?api-version=2023-06-01-preview \
-H "Content-Type: application/json" \
-H "api-key: YOUR_API_KEY" \
-H "chatgpt_url: YOUR_RESOURCE_URL" \
-H "chatgpt_key: YOUR_API_KEY" \
-d \
'
{
    "dataSources": [
        {
            "type": "AzureCognitiveSearch",
            "parameters": {
                "endpoint": "'YOUR_AZURE_COGNITIVE_SEARCH_ENDPOINT'",
                "key": "'YOUR_AZURE_COGNITIVE_SEARCH_KEY'",
                "indexName": "'YOUR_AZURE_COGNITIVE_SEARCH_INDEX_NAME'"
            }
        }
    ],
    "messages": [
        {
            "role": "user",
            "content": "What are the differences between Azure Machine Learning and Azure Cognitive Services?"
        }
    ]
}
'
```


## 「Add your data」から自動デプロイしたWeb Appsに対して、APIリクエストをするパターン
こちらについて少し詳しくみていきます。
まず、今回は説明を簡単にするために既定で付いているAzure AD認証を外します。
Web Appsの[認証]から構成済みのAAD認証を外して認証を削除します。
![](/images/azure-openai-add-your-data-api/3.png)
![](/images/azure-openai-add-your-data-api/4.png)


もし認証をつけたままリクエストしたい場合はここらへんを参考にJWTを取得してリクエストすればいける（はず）。
https://dev.classmethod.jp/articles/azure-ad-id-token-oidc/

https://learn.microsoft.com/ja-jp/azure/active-directory/develop/access-tokens


そして、こんな感じでリクエストします。
```
curl --request POST \
  --url https://{your web apps}.azurewebsites.net/conversation \
  --header 'Content-Type: application/json' \
  --data '{
        "messages": [
            {"role": "user", "content": "I want to rent a luxurious house."}
        ]
    }'
```

そうすると（見づらい）ですが、回答が返ってきます。よくみると独自ナレッジのリファレンスとなるような情報も返ってきていることが見て取れます。この場合もレスポンス形式は[ドキュメント](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#example-response-3)に記載されている通りです。
![](/images/azure-openai-add-your-data-api/5.png)


# まとめ
`Completions extensions APIを直接叩くパターン`はリクエストごとにデータソースを切り替えることができるので、会話の内容によってデータソースを切り替えたり柔軟に利用できます。反面、リクエストボディが煩雑になることによるローコード/ノーコードでの利用や、データーソースを固定した形での社内APIのような形での提供方式には向かないかもしれません。

`「Add your data」から自動デプロイしたWeb Appsに対して、APIリクエストをするパターン`はクラアントからのリクエストがシンプルになります。一方でWebAppsをデプロイする必要があるため、その分のコストがかかったり、システム全体における障害発生ポイントが1つ増えるといったリスクがあります。

それぞれの方式で一長一短あると思うので用途にあった方をお使いください。

# 公式教材（無料）
Microsoftが提供しているEラーニングサイト「MS Learn」にて、Azure Open AIのAdd your dataに関する無料教材が公開されています。
Azureアカウントを持っていなくてもサンドボックス環境でハンズオンを行うことができますので、ぜひご活用ください。
https://learn.microsoft.com/en-us/training/modules/use-own-data-azure-openai/