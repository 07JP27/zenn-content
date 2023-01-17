---
title: "Azure App Configuration参照を使ってみよう"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "appconfiguration", "azurefunctions", "webapps"]
published: true
publication_name: "microsoft"
---

:::message
この記事で紹介している内容は記事執筆時点ではプレビュー機能として提供されています。
:::

:::details TL;DR
Azure KeyVault参照のように以下の構文でApp Configurationの構成値もAzure FunctionsとApp Serviceで参照できるようになりました。
`@Microsoft.AppConfiguration(Endpoint=https://myAppConfigStore.azconfig.io; Key=myAppConfigKey; Label=myKeysLabel)​`
:::

# はじめに
[Azure App Configuration](https://docs.microsoft.com/ja-jp/azure/azure-app-configuration/)はアプリケーション設定をKey-Valueで保存し、アプリケーションへ配信する構成ストアと呼ばれるサービスです。

今まではApp Configurationに保存された構成値をクライアントアプリやWebアプリ内からコードによって構成を読み取ることができましたが、今回のアップデートによって環境変数を介した構成の参照ができるようになりました。
似たような機能としてシークレットや証明書などを保管しておく[Azure KeyVaultのKeyVault参照](https://docs.microsoft.com/ja-jp/azure/app-service/app-service-key-vault-references?tabs=azure-cli)がありますが、それのApp Configuration版と考えればよさそうです。
https://azure.microsoft.com/ja-jp/updates/public-preview-app-configuration-references-for-app-service-and-azure-functions/
![](/images/azure-app-configuration-ref/overview.png)

この機能を使うことによって以下のような利点が考えられます。
- 環境変数を介することによりApp Congifuration固有のコーディングが必要なくなるため、アプリケーションの可搬性が向上します。


なお、今回App Configuration参照がサポートされたのは以下の２つのサービスです。
- Azure Functions
- App Service

# 使ってみる
今回はAzure Functionsを使ってApp Configuration参照を試してみます。

## Azure Functions の準備
Azure Functionsを作成し、HTTPトリガーの関数を追加します。（今回は機能を試すだけなのでポータル上で開発します）
![](/images/azure-app-configuration-ref/1.png)

HTTPリクエストを受けたら`Message`という名前の環境変数を読み取って返すだけのプログラムを書いて保存をします。
``` csharp
public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    string responseMessage = Environment.GetEnvironmentVariable("Message");
    return new OkObjectResult(responseMessage);
}
```
Azure Functionsの`構成`から`新しいアプリケーション設定`を追加し名前を`Message`、値には適当な文字列を入れて保存します。（ここを後でApp Configuration参照に変更します）
![](/images/azure-app-configuration-ref/2.png)

関数のページに戻ってアクセス用のURLを取得してブラウザからアクセスします。
![](/images/azure-app-configuration-ref/3.png)
固定値で設定した値が表示されたらAzure Functionsの関数の準備は完了です。
![](/images/azure-app-configuration-ref/4.png)

## マネージドIDの有効化
Azure FunctionsがApp Configurationを参照する際に使用するIDを有効化します。
Azure Functionsの`ID`から`システム割り当て済み`の`状態`をオンにして保存します。
![](/images/azure-app-configuration-ref/5.png)

## App Configurationの準備
App Configurationを作成します。
`Configuration explorer`から`Create`→`Key-value`で新しい構成値を作成します。
![](/images/azure-app-configuration-ref/6.png)

## App ConfigurationへマネージドIDの割り当て
Azure Functionsで作成したマネージドIDをApp Configurationへアクセス付与をします。
![](/images/azure-app-configuration-ref/7.png)
今回は参照しかしないのでロールを`App Configurationデータ閲覧者`に設定します。
![](/images/azure-app-configuration-ref/8.png)
Azure Functionsで有効化したマネージドIDを選択します。（数が多い場合はフィルタなどで絞り込んでください）
![](/images/azure-app-configuration-ref/9.png)
設定内容が正しいことを確認して`割り当て`をします。
![](/images/azure-app-configuration-ref/10.png)


## Azure FunctionsでApp Configuration を追加
`Access keys`や`概要`ページからApp Configurationのエンドポイントをコピーしておきます。
![](/images/azure-app-configuration-ref/11.png)

Azure Functionsの`構成`ページから先ほど固定値を入れた`Message`をペンマークをクリックして編集します。
![](/images/azure-app-configuration-ref/12.png)



編集画面で`値`に以下のように参照構文を記述します。
```
@Microsoft.AppConfiguration(Endpoint=https://{your-endpont}.azconfig.io; Key=message)​
```
![](/images/azure-app-configuration-ref/13.png)

App ConfigurationでLabelなどを指定した場合も参照構文のパラメーターとして指定できます。
詳細な構文の解説は[公式Docs](https://docs.microsoft.com/en-us/azure/app-service/app-service-configuration-references#reference-syntax)を参照してください。

`保存`をクリックします。保存が成功するとアプリケーションが再起動します。
![](/images/azure-app-configuration-ref/14.png)

関数のURLにアクセスしてみて、App Configurationに設定された値が表示されていれば成功です！
![](/images/azure-app-configuration-ref/15.png)



# 最後に
プレビュー時点では以下のような制限があります。
- プライベートエンドポイントなどネットワークで制限されたApp Configurationの参照は不可です。
- App ConfigurationにKey Vault参照を追加して、さらにそれをFunctionsなどで参照する二次的な参照の解決はサポートさてれいません。
- 参照された構成値がApp Configurationで更新された場合でも自動的に更新されず、アプリが再起動したときにのみ新しい構成地が反映されます。
    :::message alert
    私が試したときは構成を変更して`保存`したときに実行される再起動では正常に反映されましたが、Azure Functionsの`概要`ページの`再起動`ボタンのクリックでは反映されませんでした。現在はプレビューなので後々修正されるかもしれません。
    :::