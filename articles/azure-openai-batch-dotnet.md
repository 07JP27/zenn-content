---
title: "Azure OpenAI .NET SDKでBatch処理を実行する"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai", "batch", "dotnet"]
published: true
publication_name: "microsoft"
---


単純なSDKの使用方法の解説ですが、公式ドキュメントや第三者のブログが見当たらなかったため、本記事にまとめます。

## 前提：.NET SDKのBatch周りはまだ成熟していない（2025年9月時点）
Azure OpenAIのBatch機能自体はすでにGAしていますが、.NET SDKのBatch関連の実装は以下のような点から、まだ成熟していない印象を受けます。

- Batch処理をハンドリングするBatchClientを使用しようとすると「OPENAI001：Type is for evaluation purposes only and is subject to change or removal in future updates. Suppress this diagnostic to proceed.」の警告が表示され、抑制して使用する必要がある。
- メソッドなどのインターフェイスがバイナリ形式でやりとりする部分が多く、型安全性の恩恵を受けにくい。

なお、Azure OpenAI .NET SDKのBatch部分のコードは本家OpenAIの.NET SDKを内部的に参照しているため、このあたりの事情は本家のSDKを利用する場合も同様です。
サービスとして提供する本番環境などで使用する場合は、REST APIを直接利用するコードを自前で実装するなどの対応を検討したほうが良いかもしれません。

一方、ちょっと試してみたい場合や自作ツールに組み込む程度であれば、.NET SDKを使うのもありだと思います。
OPENAI001は抑制しておき、変更があればその都度対応すればよいですし、バイナリでやりとりする部分も少し手間ですが自分でDTOクラスを定義して変換すれば型安全に扱うこともできます。

## 処理の流れ
Batch処理は、一般的に広く使われるチャット補完とは異なり、ステートフルでやや癖のある処理になります。
具体的な処理の流れは以下の通りです。なお、この流れは.NET SDKに限らず、REST APIを直接利用する場合も同様です。

1. JSONL形式の入力データをアップロード
2. 1のファイルを指定してバッチジョブを作成
3. 2で作成したジョブの完了を待つ(ポーリング)
4. ジョブの結果をJSONL形式でダウンロード

参考（公式ドキュメント）：[Azure OpenAI Batch デプロイの入門](https://learn.microsoft.com/ja-jp/azure/ai-foundry/openai/how-to/batch?tabs=global-batch%2Cstandard-input%2Cpython-secure&pivots=rest-api#upload-batch-file)

## .NET SDKを使って実装してみる
では、前節の各ステップのポイントとなるコードを見ていきます。
コンソールアプリ形式のフルサンプルコードは以下のGistに掲載しているので、必要に応じてご参照ください。
https://gist.github.com/07JP27/a1d214125d7c47f75d64c87bf20dc74b


### 1. JSONL形式の入力データをアップロード
バッチジョブを実行するためには、まずJSONL形式の入力データをAzure OpenAIにアップロードする必要があります。
データのアップロードには`OpenAIFileClient`を使用します。

`requestPayload`の型である`BatchRequestItem`とその内部で使用している型は[公式ドキュメントのJSONLの構造](https://learn.microsoft.com/ja-jp/azure/ai-foundry/openai/how-to/batch?tabs=global-batch%2Cstandard-input%2Cpython-secure&pivots=rest-api#input-format)を参考に独自にレコードを定義しています。

```csharp
OpenAIFileClient fileClient = azureClient.GetOpenAIFileClient();

var requestPayload = new BatchRequestItem[]
{
    new(
        Body: new BatchRequestBody(
            Model: "gpt-4.1-batch", // Batch用にデプロイしたデプロイ名を指定
            Messages:
            [
                new ChatMessage(Role: "system", Content: "You are a helpful assistant. Answer in Japanese."),
                new ChatMessage(Role: "user", Content: "Azure OpenAIのBatchの特徴を一文で説明して。")
            ]
        )
    ),
    new(
        .....
    )
};

string jsonlContent = string.Join("\n", requestPayload.Select(item => JsonSerializer.Serialize(item)));
using MemoryStream fileStream = new MemoryStream(System.Text.Encoding.UTF8.GetBytes(jsonlContent));

OpenAIFile uploadedFile = await fileClient.UploadFileAsync(
    fileStream,
    "batch_input.jsonl",
    FileUploadPurpose.Batch
);
```

`uploadedFile`にはアップロードしたファイルのIDなどが含まれています。
これを実行した時点で、Azure AI FoundryのAzure OpenAI Studioでもファイルがアップロードされたことが確認できます。
![入力ファイルの確認](/images/azure-openai-batch-dotnet/1.png)

### 2. 1のファイルを指定してバッチジョブを作成
ここで`BatchClient`と`CreateBatchOperation`がOPENAI001の警告対象となっているため、抑制しています。
```csharp
#pragma warning disable OPENAI001
BatchClient batchClient = azureClient.GetBatchClient();
#pragma warning restore OPENAI001

var batchRequest = new
{
    input_file_id = uploadedFile.Id,
    endpoint = "/chat/completions",
    completion_window = "24h",
    output_expires_after = new
    {
        seconds = 1209600
    },
    anchor = "created_at"
};

BinaryContent content = BinaryContent.Create(
    BinaryData.FromObjectAsJson(batchRequest)
);

#pragma warning disable OPENAI001
CreateBatchOperation result = await batchClient.CreateBatchAsync(content, false);
#pragma warning restore OPENAI001
```
ここまで実行した時点で、Azure AI FoundryのAzure OpenAI Studioでもバッチジョブが作成されたことが確認できます。
![バッチジョブの確認](/images/azure-openai-batch-dotnet/2.png)

なお、`CreateBatchAsync`の第2引数は`waitForCompletion`ですが、trueにするとジョブの完了まで待機できるようです（今回は動作未検証）。

### 3. 2で作成したジョブの完了を待つ(ポーリング)

定期的なポーリング（公式ドキュメントでは[60秒以上が推奨](https://learn.microsoft.com/ja-jp/azure/ai-foundry/openai/how-to/batch?tabs=global-batch%2Cstandard-input%2Cpython-secure&pivots=rest-api#track-batch-job-progress:~:text=%E5%90%84%E7%8A%B6%E6%85%8B%E5%91%BC%E3%81%B3%E5%87%BA%E3%81%97%E3%81%AE%E9%96%93%E3%81%AB%E5%B0%91%E3%81%AA%E3%81%8F%E3%81%A8%E3%82%82%2060%20%E7%A7%92%E5%BE%85%E6%A9%9F%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%82%92%E3%81%8A%E5%8B%A7%E3%82%81%E3%81%97%E3%81%BE%E3%81%99%E3%80%82)）でステータスを確認し、完了するまで待ちます。
ステータス一覧はREST APIドキュメントの「[バッチ ジョブの進行状況を追跡する](https://learn.microsoft.com/ja-jp/azure/ai-foundry/openai/how-to/batch?tabs=global-batch%2Cstandard-input%2Cpython-secure&pivots=rest-api#track-batch-job-progress)」に記載されている値を参考にハンドリングしています。

バッチジョブを作成したときに返却された`CreateBatchOperation`クラスに`GetBatchAsync`メソッドが用意されているので、これを使用してステータスを取得します。
ポーリングを行うことにより、全体のうちどれくらいの数が完了したかが取得できるので、進捗表示などに活用できます。

```csharp
BatchStatus currentStatus = new BatchStatus();

while (currentStatus.Status != "completed" &&
       currentStatus.Status != "failed" &&
       currentStatus.Status != "cancelled" &&
       currentStatus.Status != "expired")
{
    ClientResult statusResult = await result.GetBatchAsync(null);
    BinaryData statusData = statusResult.GetRawResponse().Content;

    currentStatus = JsonSerializer.Deserialize<BatchStatus>(statusData) ?? throw new Exception("Failed to deserialize batch status.");

    if (currentStatus.RequestCounts != null)
    {
        var counts = currentStatus.RequestCounts;
        int total = counts.Total ?? 0;
        int completed = counts.Completed ?? 0;
        int failed = counts.Failed ?? 0;

        Console.WriteLine($"[{DateTime.Now:HH:mm:ss}] Status: {currentStatus.Status}");
        Console.WriteLine($"  Progress: {completed}/{total} completed, {failed} failed");
    }
    else
    {
        Console.WriteLine($"[{DateTime.Now:HH:mm:ss}] Status: {currentStatus.Status}");
    }
    await Task.Delay(TimeSpan.FromSeconds(60));
}
```

これを実行すると、以下のように進捗が表示されます。
```
...
[13:06:30] Status: in_progress
  Progress: 0/2 completed, 0 failed
[13:07:30] Status: in_progress
  Progress: 0/2 completed, 0 failed
[13:08:31] Status: in_progress
  Progress: 0/2 completed, 0 failed
[13:09:31] Status: in_progress
  Progress: 2/2 completed, 0 failed
[13:10:31] Status: in_progress
  Progress: 2/2 completed, 0 failed
[13:11:32] Status: in_progress
  Progress: 2/2 completed, 0 failed
[13:12:32] Status: in_progress
  Progress: 2/2 completed, 0 failed
[13:13:32] Status: finalizing
  Progress: 2/2 completed, 0 failed
[13:14:33] Status: completed
  Progress: 2/2 completed, 0 failed
```
このログから、2件の推論リクエストがすべて完了しても、ジョブ自体が即座に完了状態になるわけではないことがわかります。

### 4. ジョブの結果をJSONL形式でダウンロード

完了したジョブの結果をダウンロードするには`BatchClient`ではなく、入力ファイルと同じように`OpenAIFileClient`を使用して、JSONL形式でダウンロードします。
結果には推論の応答だけでなく、どれくらいのトークンを使ったかなどのメタ情報も含まれているため、コスト管理などにも役立ちます。
```csharp
BinaryData outputContent = await fileClient.DownloadFileAsync(currentStatus.OutputFileId);

string jsonlOutput = outputContent.ToString();
string[] lines = jsonlOutput.Split('\n', StringSplitOptions.RemoveEmptyEntries);

Console.WriteLine("\n========== Batch Processing Results ==========");
foreach (string line in lines)
{
    BatchOutputItem? outputItem = JsonSerializer.Deserialize<BatchOutputItem>(line);
    if (outputItem != null)
    {
        Console.WriteLine($"\nCustom ID: {outputItem.CustomId}");
        Console.WriteLine($"Status Code: {outputItem.Response?.StatusCode}");
        
        if (outputItem.Response?.Body?.Choices != null && outputItem.Response.Body.Choices.Length > 0)
        {
            var firstChoice = outputItem.Response.Body.Choices[0];
            Console.WriteLine($"Response: {firstChoice.Message?.Content}");
        }
        
        if (outputItem.Error != null)
        {
            Console.WriteLine($"Error: {outputItem.Error}");
        }
    }
}
Console.WriteLine("====================================\n");
```

結果として以下が出力されます。なお、順序は必ずしも入力順とは限らないようなので、順序保証が必要な場合は`Custom ID`などの値でソートするなどの対応が必要そうです。
```
========== Batch Processing Results ==========

Custom ID: fddab681-aad5-4ef3-8b22-918b3189b303
Status Code: 200
Response: Azure OpenAIのBatchは大量のリクエストをまとめて非同期で処理できるのに対し、通常のCompletionは1リクエストごとに同期的に応答を返します。

Custom ID: 14e5da3d-cbb5-4af7-9327-86f5523ddc2f
Status Code: 200
Response: Azure OpenAIのBatchは、大量の入力データに対して効率的に並列処理を行い、複数のリクエストをまとめて一括で推論できる機能です。
====================================
```

### 4. 後処理
アップロードした入力ファイルは不要なので削除しておきます。
```csharp
await fileClient.DeleteFileAsync(fileId);
// ジョブは有効期限を設定して作成しているので勝手に消えるが、ここで明示的に消しても問題ない
```

再掲になりますが、コンソールアプリ形式のフルサンプルコードは以下のGistに掲載しているので、必要に応じてご参照ください。
https://gist.github.com/07JP27/a1d214125d7c47f75d64c87bf20dc74b


## 参考
https://github.com/openai/openai-dotnet

https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.OpenAI_2.3.0-beta.2/sdk/openai/Azure.AI.OpenAI/src
