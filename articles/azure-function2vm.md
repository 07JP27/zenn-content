---
title: "Azure FunctionsからAzureファイル共有を介して仮想マシンへファイルを連携する"
emoji: "📬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azurefunctions", "azure"]
published: true
publication_name: "microsoft"
---

# はじめに
クラウドネイティブなアーキテクチャを構成していても、使用するミドルウエアなどの制限でVM上で処理を実行しなければいけないということは現実的にあり得ると思います。
そこで今回は下図のようにAzure FunctionsとAzuer VMをAzureファイル共有を介して連携する方法を紹介します。
![](/images/azure-function2vm/overview.png)


# リソースの作成
次のリソースを作成しておきます。
- ストレージアカウント
    - Azure ファイル共有 
- Azure 仮想マシン
    - Windows(Linuxでも可)
- Azure Functions
    - .NET
# Azure VMへのAzureファイル共有のマウント
作成したファイル共有をAzure VMにマウントします。
詳しくは以下のドキュメントにステップバイステップで記載がありますが、ファイル共有の「接続」を選択してドライブ文字を選択してマウント用のスクリプトをコピーします。ここでLinuxにマウントする場合は「Linux」タブを選択してスクリプトをコピーします。

https://learn.microsoft.com/ja-jp/azure/storage/files/storage-how-to-use-files-windows

![](/images/azure-function2vm/mountscript.png)

Azure VMにリモートデスクトップで接続して、スクリプトを実行します。実行後にファイルエクスプローラーを確認するとドライブ（この場合はZ）がマウントされています。
![](/images/azure-function2vm/mounted.png)

現在は何もファイルを入れていないのでドライブ内は空です。
![](/images/azure-function2vm/initdrive.png)

# Azure Functionsのコーディング
Azure FilesのSDKを使用してファイルをアップロードする処理を実装します。
「FromFunctions」というフォルダを作成して、その中に適当なテキストに日時を付加してテキストファイルを作るような処理です。

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using System.Security.Cryptography.X509Certificates;
using Azure.Storage.Files.Shares;
using System.Text;
using Azure;

namespace Injest2Fileshare
{
    public static class Function1
    {
        [FunctionName("Injest2Fileshare")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            // 本番運用ではKeyVaultや環境変数を使用して隠ぺいしてください。可能である場合はマネージドIDかSASキーの使用を推奨します。
            string connectionString = "{接続文字列}";
            string shareName = "myfileshare(ご自身のファイル共有名)";

            ShareClient share = new ShareClient(connectionString, shareName);

            if (await share.ExistsAsync())
            {
                ShareDirectoryClient directory = share.GetDirectoryClient("FromFunctions");
                await directory.CreateIfNotExistsAsync();
                if (await directory.ExistsAsync())
                {
                    string fileName = DateTime.Now.ToString("yyyyMMddHHmmss") + ".txt";
                    ShareFileClient file = directory.GetFileClient(fileName);

                    string content = "This is a sample text."+ Environment.NewLine + DateTime.Now.ToString();
                    byte[] buffer = Encoding.UTF8.GetBytes(content);
                    long fileSize = buffer.Length;
                    file.Create(fileSize);

                    using (MemoryStream stream = new MemoryStream(buffer))
                    {
                        file.UploadRange(
                            new HttpRange(0, stream.Length),
                            stream);
                    }
                }
            }

            return new OkResult();
        }
    }
}

```

# テスト実行
Azure PortalでAzure Functionsの「コードとテスト」からテスト実行をします。
![](/images/azure-function2vm/testexec.png)

Azure VMのドライブを確認すると「FromFunctions」フォルダとテキストファイルが作成されており、ファイルの中身もAzure Functionsで指定した文字列が入っていることが確認できます。
![](/images/azure-function2vm/result.png)

これでAzureファイル共有を介してAzure FunctionsからAzure VMへファイルを連携することができました。
