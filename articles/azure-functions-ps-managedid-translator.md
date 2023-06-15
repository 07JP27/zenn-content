---
title: "Azure FunctionsのPowerShellからマネージドIDを使ってCognitive Translatorを呼んでみよう"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "powershell", "azurefunctions"]
published: true
---

※本記事で使用するAzure Cognitive Services内の翻訳リソースの正式な名称は「Translator」ですが、他の翻訳サービスと混同しないように「Cognitive Translator」と表記します。

# はじめに
タイトル通りのことをしようとしたのですが、断片的な情報しかなかったため、まとめておきます。アーキテクチャ概要はこんな感じです。

![](/images/azure-functions-ps-managedid-translator/architecture.png)

C# SDKなどであれば[DefaultAzureCredential](https://learn.microsoft.com/ja-jp/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet)クラスが用意されているので、これを使うことによって意識せずにコードからマネージド ID を使用することができますが、PowerShellから呼び出す場合は少し手順が必要です。

# やってみる
## リソースデプロイ
Azure FunctionsとCognitive Translatorをデプロイします。
ここは通常通りのリソースデプロイなので詳細な説明は省略します。

## Azure Fucntions のマネージド ID を有効化
Azure FunctionsのマネージドIDを有効化します。
Azure Portalで、デプロイした**Azure Functionsにアクセスして**[`ID`]→[`システム割り当て済み`]→[`オン`]にして[`保存`]します。
割り当てが成功すると[`オブジェクト(プリンシパル)ID`]が表示されるはずです。マネージドIDの割り当ての確認で使用するのでメモしておきましょう。
![](/images/azure-functions-ps-managedid-translator/1.png)

## Cognitive Translator へのアクセス権限を付与
Azure FunctionsのマネージドIDにCognitive Translatorへのアクセス権限を付与します。
Azure Portalで、デプロイした**Cognitive Translatorにアクセスして**[`アクセス制御(IAM)`]→[`ロールの割り当ての追加`]を選択します。
![](/images/azure-functions-ps-managedid-translator/2.png)

Azure Functionsへ与えるロールを選択します。単純に翻訳をするだけであれば[`Cognitive Services ユーザー`]を選択すればOKです。[`次へ`]を選択。
![](/images/azure-functions-ps-managedid-translator/3.png)

割り当て対象のリソースを選択します。[`アクセスの割り当て先`]で[`マネージドID`]を選択して[`メンバーを選択する`]からマネージドIDを有効化したAzure Fucnctionsを選択して[`選択`]→[`レビューと割り当て`]を選択します。
![](/images/azure-functions-ps-managedid-translator/4.png)

確認画面が表示されます。ここで

- ロールが [`Cognitive Services ユーザー`]になっていること
- メンバーのオブジェクトIDがAzure FunctionsのマネージドIDと一致していること

を確認して[レビューと割り当て]を選択します。
![](/images/azure-functions-ps-managedid-translator/5.png)

## Cognitive Translator の情報をメモ
APIリクエストで必要な情報を収集します。
Azure Portalで、デプロイした**Cognitive Translatorにアクセスして**[`キーとエンドポイント`]→[`場所/地域`]をメモしておきます。
![](/images/azure-functions-ps-managedid-translator/6.png)
同様に[`プロパティ`]→[`リソースID`]をメモしておきます。
![](/images/azure-functions-ps-managedid-translator/7.png)

## コーディング
Azure Portalで、デプロイした**Azure Functionsにアクセスして**[`関数`]→[`作成`]から適当な関数を作成します。本記事では HTTP triggerで作成しています。

```powershell
using namespace System.Net
param($Request, $TriggerMetadata)
Write-Host "PowerShell HTTP trigger function processed a request."

# Managed IDからアクセストークンを取得
$resourceURI = "https://cognitiveservices.azure.com/"
$tokenAuthURI = $env:IDENTITY_ENDPOINT + "?resource=$resourceURI&api-version=2019-08-01"
$tokenResponse = Invoke-RestMethod -Method Get -Headers @{"X-IDENTITY-HEADER"=$env:IDENTITY_HEADER} -Uri $tokenAuthURI
$accessToken = $tokenResponse.access_token

# Cognitive Translatorに翻訳リクエスト
$baseUri = "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0"
$headers = @{
    'Ocp-Apim-Subscription-Region' = '{メモしたCognitive Translatorの場所/地域}'
    'Ocp-Apim-ResourceId' = '{メモしたCognitive TranslatorのリソースID}'
    'Content-type' = 'application/json'
    'Authorization' = 'Bearer '+$accessToken
}

Write-Host "Header:"$headers

$textJson = @{
    "Text" = "こんにちは"
} | ConvertTo-Json

$body = "[$textJson]"
$uri = "$baseUri&to=en"

$result = Invoke-RestMethod -Method Post -Uri $uri -Headers $headers -Body $body -ContentType "application/json; charset=utf-8"
$translatedText = $result.translations.text

# HTTPトリガーのレスポンスを返す
Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = [HttpStatusCode]::OK
    Body = $translatedText
})
```
※本質的な部分を説明するために、エラーハンドリングやリトライ処理などは省略しています。

**重要なポイントをいくつか挙げます**

- 環境変数 IDENTITY_ENDPOINT、IDENTITY_HEADERにはマネージドIDをオンにすると自動で値が設定されるのでIDENTITY_ENDPOINTに対してトークン要求をする。： [参考](https://learn.microsoft.com/ja-jp/azure/app-service/overview-managed-identity?tabs=portal%2Chttp#rest-endpoint-reference)
- resource に設定するのは「`https://cognitiveservices.azure.com/`」。：[参考](https://learn.microsoft.com/ja-jp/azure/cognitive-services/authentication?tabs=powershell#sample-request)

- Cognitive TranslatorでマネージドIDを使用する場合、Ocp-Apim-ResourceIdヘッダーに認証対象のCognitive TranslatorのリソースIDを指定する必要がある。：[参考](https://learn.microsoft.com/ja-jp/azure/cognitive-services/translator/reference/v3-0-reference#authentication-with-azure-active-directory-azure-ad)

## テスト

それでは実際にテストをしてみます。
コーディングをした画面の[`テストと実行`]→[`実行`]をクリックしてみましょう。
出力タブに200の応答と「こんにちは」が英語に翻訳されたものが表示されます。
![](/images/azure-functions-ps-managedid-translator/8.png)

これで API キーを使うことなくCognitive Translatorを使用することができました。

# まとめ
マネージドIDは認証基盤として非常に強力な一方、実装方法はSDKや言語によって異なるため、バッチリ合うドキュメントが見つからない場合があります。今回のコードを改変して実際のシナリオに合わせていただければ幸いです。

# 断片的な参考情報
この記事を書くにあたって参考にした情報たちです。

https://learn.microsoft.com/ja-jp/azure/app-service/overview-managed-identity?tabs=portal%2Chttp

https://learn.microsoft.com/ja-jp/azure/cognitive-services/authentication?tabs=powershell#sample-request

https://learn.microsoft.com/ja-jp/azure/cognitive-services/translator/reference/v3-0-reference#authentication-with-azure-active-directory-azure-ad
