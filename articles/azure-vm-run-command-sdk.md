---
title: "Azure .NET SDKでAzure VMに対してRun Commandを実行する"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'dotnet', 'vm', 'runcommand']
published: true
publication_name: "microsoft"
---

## Run Commandとは
そもそもRun Commandとはなにか、という話ですが、一言で言えば**Azure VMに対して外部からコマンドを実行する機能**です。
VM内からコマンドプロンプトなどでコマンドが実行できるのは当然ですが、Run Commandを使うとAzure VMにインストールされた拡張機能を介して外部からコマンドが実行できます。
これにより、VMにログインすることなく外部からVM内部の情報にアクセスできるため、大量のリソース管理などに便利です。
Run CommandはAzure PortalやAzure CLI、Azure PowerShellなどから実行できますが、Azure .NET SDKでも実行することができます。

https://learn.microsoft.com/ja-jp/azure/virtual-machines/run-command-overview

## Azure .NET SDKでRun Commandを実行する
いくつかのパッケージをインストールすふ必要がありますが、それほど複雑な処理ではありません。
当たり前ですが、対象のVMが起動していることが前提条件です。起動していない場合、Run Commandを実行するとエラーが発生します。

```csharp
using Azure;
using Azure.Core;
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.Compute;
using Azure.ResourceManager.Compute.Models;

// vm取得用のARMクライアントを作成。トークンの取得方法はいくつかあるが、今回はDefaultAzureCredential認証情報を使う
// ref:https://learn.microsoft.com/ja-jp/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet
var client = new ArmClient(new DefaultAzureCredential());

// 対象のVMを一意に指定
ResourceIdentifier target = new ResourceIdentifier("/subscriptions/xxxxxxxxx-xxxxx-xxxx-xxxxx-xxxxxxxx/resourceGroups/{rgname}/providers/Microsoft.Compute/virtualMachines/{vmname}");

// リソースを取得
var vm = await client.GetVirtualMachineResource(target).GetAsync();

// PowerShellで実行する場合
// 他の選択肢はOSによって異なる：https://learn.microsoft.com/ja-jp/azure/virtual-machines/windows/run-command#available-commands
var commnad = new RunCommandInput("RunPowerShellScript");

// 実行するコマンドを追加
commnad.Script.Add("echo \"Hello World!!\"");

// Run Commandを実行 30秒~1分くらいかかる
var runCommandResult = await vm.Value.RunCommandAsync(WaitUntil.Completed, commnad);
```