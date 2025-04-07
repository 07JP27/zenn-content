---
title: "Azure FunctionsでMCPサーバーを作る！"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mcp", "azure", "azurefunctions"]
published: true
publication_name: "microsoft"
---

## 注目MCPニュース
昨今LLM界隈を賑わせているMCP（Model Context Protocol）ですが、個人的には最近のビッグニュースとして以下の２つが挙げられます。

### Azure FunctionsトリガーがMCPに対応!
Azure Functionsは「トリガー」と呼ばれる仕組みを使ってイベントドリブンなアプリケーションを構築することができます（たとえばタイマートリガーで定期実行、HTTPトリガーでWeb APIを作る、など）。
2025年4月5日にAzure FunctionsのMCPトリガーがプレビュー版として公開されたことがMicrosoftの公式ブログで発表されました。本記事執筆時点での対応言語はC#、Python、Nodeの3つです。
https://techcommunity.microsoft.com/blog/appsonazureblog/build-ai-agent-tools-using-remote-mcp-with-azure-functions/4401059

### VS Code Stable 1.99でMCPが正式サポート!
GitHub CopilotにAgent modeが追加され、それと同時にMCPががサポートされました（プレビュー）。
これまでGitHub CopilotはClineなどと比較してAgenticな動作が弱いとされていましたが、このアップデートによって業界の最先端に並んだ状況です。
https://code.visualstudio.com/updates/v1_99


というわけで今回はこの２つを掛け合わせて、Azure Functionsを使ってMCPサーバーを作り、VS CodeのGitHub CopilotのAgent modeで、そのMCPサーバーを使ってみます。


## 作るもの
今回は主にMCPサーバー（Azure Functions）の実装とMCPクライアントの疎通確認ができれば良いので、MCPサーバーの処理はあまり凝らずに、MCPのサンプルとしてよく使われる[mcp_server_time](https://github.com/modelcontextprotocol/servers/tree/main/src/time/src/mcp_server_time)と同じく現在時刻を返すだけの処理にします。

なお今回Azure Functionsの実装はC#を使用しますが、Azure FunctionsのMCPサーバートリガーは前述の通りPythonとNodeにも対応しています。

## やってみる
### MCPサーバー：Azure Functionsの作成と実行
#### Azure Functionsプロジェクトの作成
はじめに[Visual Studio Code を使用して Azure Functions を開発する](https://learn.microsoft.com/ja-jp/azure/azure-functions/functions-develop-vs-code?tabs=node-v4%2Cpython-v2%2Cisolated-process%2Cquick-create&pivots=programming-language-csharp)の「Azure Functions プロジェクトを作成する」セクションを参考にAzure Functionsのプロジェクトを作成します。

#### MCPエクステンションの追加
続いてAzure FunctionsのプロジェクトにMCPエクステンションを追加します。
パッケージは以下で公開されているため、コマンドもしくはコマンドパレット(Shift + Ctrl + P)からnugetでインストールします。
https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker.Extensions.Mcp

```bash
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.Mcp --version 1.0.0-preview.1
```
:::message
なお、pythonとNodeで実装する場合はhost.jsonにバンドル参照参照を含める必要があることに注意してください。
```json
"extensionBundle": {
  "id": "Microsoft.Azure.Functions.ExtensionBundle.Experimental",
  "version": "[4.*, 5.0.0)"
}
```
:::

#### MCPサーバーのツール実装
続いて実際のToolにあたる関数を実装します。冒頭にAzure Functionsのテンプレートを使用してHTTPトリガーの関数を作成したので、その関数を改変するのが楽です。
```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Extensions.Mcp;
using System.Globalization;
using System.Text.Json;

namespace AzfuncMcp;
public class TimeUtil
{
    [Function(nameof(GetCurrentTime))]
    public string GetCurrentTime(
        [McpToolTrigger("getcurrenttime", "Gets the current time. If no timezone is specified, the tool will return the time in UTC.")] ToolInvocationContext context,
        [McpToolProperty("timezone", "string", "The name of the timezone.")] string timezone = "UTC"
        )
    {
        try
        {
            TimeZoneInfo timeZoneInfo = TimeZoneInfo.FindSystemTimeZoneById(timezone);
            DateTime currentTime = TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, timeZoneInfo);
            
            var response = new 
            {
                timezone = timeZoneInfo.StandardName,
                time = currentTime.ToString("yyyy-MM-dd HH:mm:ss", CultureInfo.InvariantCulture),
                displayName = timeZoneInfo.DisplayName
            };
            
            return JsonSerializer.Serialize(response);
        }
        catch (TimeZoneNotFoundException)
        {
            return $"The timezone '{timezone}' was not found.";
        }
        catch (InvalidTimeZoneException)
        {
            return $"The timezone '{timezone}' is invalid.";
        }
        catch
        {
            return "Could not get the current time.";
        }
    }
}
```


#### MCPサーバーのスタートアップ処理
関数が実装できたら、次はAzure Functionsのスタートアップ処理であるProgram.csを修正してMCPツールを登録します。
```csharp Program.cs
using Microsoft.Azure.Functions.Worker.Builder;
using Microsoft.Extensions.Hosting;
using AzfuncMcp;

var builder = FunctionsApplication.CreateBuilder(args);

builder.ConfigureFunctionsWebApplication();
builder.EnableMcpToolMetadata();

builder.ConfigureMcpTool(nameof(TimeUtil.GetCurrentTime))
    .WithProperty("timezone", "string", "The timezone.");

builder.Build().Run();
```

#### Azure ストレージエミュレーター「Azurite」　の起動
特にブログ記事などでの説明が見つけられませんでしたが、MCP拡張機能はBlobストレージを使用するようです。そのためローカルで実行する場合は、AzureストレージのエミュレーターであるAzuriteを起動しておく必要があります。

まだAzuriteをインストールしたことがない場合は[Azurite をインストールする](https://learn.microsoft.com/ja-jp/azure/storage/common/storage-use-azurite?tabs=visual-studio-code%2Cblob-storage#install-azurite)を参考にインストールします。

Azuriteをインストールしたら、Shift + Ctrl + Pでコマンドパレットを開き、「Azurite: Start」を選択します。そしてAzure Functionsプロジェクトのlocal.settings.jsonでAzuriteを使用するように設定します。
```json local.settings.json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",//<=これ
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated"
  }
}
```

#### Azure Functionsの起動
では、Azure Functionsをローカルで起動して動作確認してみましょう。
```bash
func start
```
またはVS Codeのデバッグ機能を使ってデバッグ起動します。
テンプレートを使用してAzure Functionsを作成した場合、デバッグ起動のためのlaunch.jsonが自動生成されているはずです。
「Run and Debug」メニューから「Attach to .NET Functions」を選択してデバッグ起動します。

![MCPサーバーの起動](/images/azure-function-mcp/launch.png)


起動に成功するとMCPツールトリガーの関数が起動されたことが確認できます。
![MCPサーバーの起動](/images/azure-function-mcp/launched.png)

### MCPクライアント：VS CodeのGitHub Copilotの設定
Shift + Ctrl + Pでコマンドパレットを開き、「MCP: Add Server」を選択します。
![MCPサーバーの追加](/images/azure-function-mcp/addserver.png)

MCPサーバーの種類を選択します。今回はHTTP経由でAzure Functionsを使用するので、HTTPを選択します。
![MCPサーバーの種類](/images/azure-function-mcp/servertype.png)

MCPサーバーのURLを入力します。今回はローカルで起動しているので、http://localhost:7071を使用します。
MCPサーバーの拡張機能を使用すると「/runtime/webhooks/mcp/sse」のパスが自動的に生成されるので、「`http://localhost:7071/runtime/webhooks/mcp/sse`」を指定します。
![MCPサーバーのURL](/images/azure-function-mcp/serverurl.png)

MCPサーバーの名前を入力します。任意の名前でOKです。
![MCPサーバーの名前](/images/azure-function-mcp/servername.png)

MCPサーバーの追加が完了するとsettings.jsonが更新され、MCPサーバーの情報が追加されます。
もし、初期状態から何も変えていなければ、MCPサーバーのサンプルとして「"mcp-server-time」という名前のpythonのMCPサーバーがすでに存在しているはずです。今回作成したツールとの競合を避けるために、"mcp-server-time"は削除して、作成したMCPサーバーのみが登録された状態にします。

```json settings.json
//　省略
 "mcp": {
        "inputs": [],
        "servers": {
            "my-azfunc-mcp-server": {
                "type": "sse",
                "url": "http://localhost:7071/runtime/webhooks/mcp/sse"
            }
        }
    },
//　省略
```

### 疎通確認
GitHub Copilotのチャットウインドウを開きます。
この時にモードが「Agent」に設定することに注意してください。
![MCPサーバーの疎通確認](/images/azure-function-mcp/agentmode.png)

続いてMCPの設定アイコン（スパナとドライバーのアイコン）をクリックして、MCPサーバーの設定を開きます。
有効なMCPサーバーのリストが表示されるので追加したMCPサーバーが有効化されていることを確認します。
![MCPサーバーの疎通確認](/images/azure-function-mcp/serverlist.png)

そしてチャットウインドウに「今何時？」などと入力します。
すると、MCPサーバーを呼び出して良いかの確認が表示されるので、「Continue」を選択します。
![MCPサーバーの疎通確認](/images/azure-function-mcp/whattimeisit.png)

実際にMCPサーバーが呼び出され、現在時刻が表示されます。
![MCPサーバーの疎通確認](/images/azure-function-mcp/nowis.png)

なお、Azure FunctionsのデバッグウインドウにもMCPサーバーが呼び出されたことが表示されます。
![MCPサーバーの疎通確認](/images/azure-function-mcp/funclog.png)

作成したMCPサーバーは、引数にタイムゾーンを指定することができるので、例えば「シアトルは今何時？」と入力すると、自動的にシアトルのタイムゾーンが設定されMCPサーバーが呼び出されます。
![MCPサーバーの疎通確認](/images/azure-function-mcp/specifiedtimezone.png)

## まとめ
Azure Functionsは以前からLLMアプリケーションのシナリオと相性が良い場面が多かったですが、MCPサーバーにも対応したことで、よりLLMアプリケーションシナリオ適用の幅が広がりました。
MCPはまだまだ新しい技術ですが、標準のプロトコルがあり、その利用者が増えていることはエコシステムの面でも非常に良いことだと思います。

## 補足：クラウド上にデプロイする場合
ローカルで疎通確認が完了したAzure Functionsをクラウド上にデプロイする場合は、通常のAzure Functionsと同様にデプロイができます([参考情報](https://learn.microsoft.com/ja-jp/azure/azure-functions/functions-develop-vs-code?tabs=node-v4%2Cpython-v2%2Cisolated-process%2Cquick-create&pivots=programming-language-csharp#republish-project-files))。

その場合の注意点として、クラウド上のAzure Functionsはデフォルトでキー認証が設定されているため、MCPクライアントから使用する場合は設定を変更してキー情報を含める必要があります。その場合のMCPサーバーの設定（settings.json）は以下のようになります。
このように構成するとMCPサーバーを起動した際にFunctionsのキーの入力を求められ、実際にAzure Functionsの設定画面から取得したキーを入力するとクラウド上のAzure Functionsにキー認証付きで接続することができます。
```json settings.json
"mcp": {
        "inputs": [],
        "servers": {
            "my-azfunc-mcp-server": {
                "type": "sse",
                "url": "<実際にデプロイしたAzure FunctionsのベースURL>/runtime/webhooks/mcp/sse",
                "headers": {
                    "x-functions-key": "${input:functions-mcp-extension-system-key}"
                }
            }
        }
    },
```

なお、本記事の執筆時点ではWindowsのAzure Functions上ではうまく動作しないという報告があるようです(Linuxでは問題ない模様)。
https://github.com/Azure-Samples/remote-mcp-functions-dotnet/issues/8

## 参考
Azure FunctionsでのMCPサーバーの実装サンプル
https://github.com/Azure-Samples/remote-mcp-functions-dotnet/tree/main

MCPサーバー全般の実装サンプル
https://github.com/modelcontextprotocol/servers