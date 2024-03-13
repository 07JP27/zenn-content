---
title: "Azure .NET SDKでAzure Resource Graphのクロスサブスクリプションクエリを発行する"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'dotnet', 'resourcegraph']
published: true
publication_name: "microsoft"
---


```csharp
using System.Text.Json.Serialization;
using Azure.Core;
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.ResourceGraph;
using Azure.ResourceManager.ResourceGraph.Models;

// new DefaultAzureCredentialなどTokenCredential型の他の認証方法でもいい
TokenCredential cred = new AzureCliCredential(); 

ArmClient client = new ArmClient(cred);

var tenantResource = client.GetTenants().Where(x=> x.Data.TenantId == new Guid("{テナントID}")).FirstOrDefault();
ResourceQueryContent content = new ResourceQueryContent("resources| where type == \"microsoft.cognitiveservices/accounts\" | project id"){};
ResourceQueryResult result = await tenantResource.GetResourcesAsync(content);

var resultRecords = result.Data.ToObjectFromJson<List<RGRecord>>();

foreach (var record in resultRecords)
{
    Console.WriteLine($"Id: {record.Id}");
}

;
class RGRecord
{
    [JsonPropertyName("id")]
    public string? Id { get; set; } = null;
}
```

上記を実行すると環境にもよりますが、指定したテナント配下のすべてのサブスクリプションに存在するCognitive ServicesのリソースのIDが取得できます。
```bash
Id: /subscriptions/621a9181-xxx-xxx-xxx-91e28edfa0b4/resourceGro...
Id: /subscriptions/803673f4-xxx-xxx-xxx-1d9ca8b2cfa1/resourceGro...
Id: /subscriptions/fd504c94-xxx-xxx-xxx-d84d80895457/resourceGro...
Id: /subscriptions/fd504c94-xxx-xxx-xxx-d84d80895457/resourceGro...
Id: /subscriptions/fd504c94-xxx-xxx-xxx-d84d80895457/resourceGro...
Id: /subscriptions/fd504c94-xxx-xxx-xxx-d84d80895457/resourceGro...
...
```


ちなみにサブスクリプションを指定する場合はクエリオブジェクトの生成を以下のようにします。
```csharp
ResourceQueryContent content = new ResourceQueryContent("resources| where type == \"microsoft.cognitiveservices/accounts\" | project id")
{
    Subscriptions =
    {
        "621a9181-xxx-xxx-xxx-91e28edfa0b4"
    },
};
```