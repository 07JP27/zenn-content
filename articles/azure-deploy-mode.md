---
title: "Azureのデプロイモードの違いと使い所"
emoji: "🏗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "arm", "bicep"]
published: true
---

# デプロイモード
Azureでリソースをデプロイする際はAzure PortalやAzure CLIなど、どのようなUIを使用しても、Azure Resource Manager(ARM)と呼ばれる管理サービスを通してデプロイ（変更・削除を含む）が行われます。

![](/images/azure-deploy-mode/arm.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/management/overview)

このときのオプションに**デプロイモード**という以下の２つのオプションがあります。
- 増分モード（Incremental mode）
- 完全モード（Complete mode）

本記事ではこれらのデプロイモードについて違いと使い所を整理します。

# 各モードの挙動について
最大の違いは「**リソースグループには存在するが、テンプレートに定義されていないリソースに対しての挙動**」です。いくつか細かい違いもあるので具体的に見ていきましょう。
## 増分モード（Incremental mode）
増分モードは、一般的にイメージしやすい挙動です。リソースグループには存在するが、テンプレートに定義されていないリソースは**変更せず、そのまま残します**。 名前の通り追加のみを行うモードです。
テンプレートに定義されたリソースは、リソースグループに追加され、変更されているものは**プロパティが再適用**されます。
**増分モードは、デプロイモードの規定値**であり、特にモードを意識せずにデプロイを行なっている場合は増分モードになります。
![](/images/azure-deploy-mode/incremental.png)

:::message
イメージしやすい増分モードですが、リソースの「更新」は注意が必要です。
既存のリソースを増分モードで再デプロイすると、すべてのプロパティが再適用されます。増分追加ではありません。
つまり、「現在のSKUから変更がないのでテンプレートでSKUを省略する」といったことは**できません**
:::

## 完全モード（Complete mode）
完全モードは、リソースグループには存在するがテンプレートに定義されていないリソースを**削除します**。完全モードを使用する場合はデプロイ時に明示的に指定する必要があります。
なお、完全モードでの以下のデプロイはサポートされていません。
- Azure Portalを使用したデプロイ
- デプロイのスコープをサブスクリプションに設定する「[サブスクリプションレベルのデプロイ](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/templates/deploy-to-subscription?tabs=azure-cli)」
![](/images/azure-deploy-mode/complete.png)

# 完全モードは何のためにあるのか？
ざっくり一言で言うと「**IaC(Infrastructure as Code)のため**」です。
IaCは宣言型の構成ファイルでリソースの「あるべき姿」を宣言し、それをクラウド基盤(Azureの場合はARM)に与えることで宣言されたリソースがデプロイされます。
その際に手動で追加されたリソースが存在する場合、テンプレートに定義されたあるべき姿とは違った状態になってしまいます。それを防ぎ「テンプレートにあるものはある、ないものはない」状態を生み出し、**テンプレートの冪等性[^1]を担保する**のが完全モードと言えます。


なお、IaCのための宣言型ファイルはAzure固有だとBicepやARMテンプレート、マルチクラウドだとTerraformが有名どころです。


# 完全モードを使用する際の注意点
## What-Ifを使う
完全モードでのデプロイはリソースが削除されるリスクがあるため、**デプロイ前にWhat-If操作をすることが強く推奨されています。**（ドキュメント上は「常に使用してください」という表記）
What-Ifを使うとテンプレートをデプロイした場合にリソースがどのように変更されるかを確認できます。具体的なWhat-Ifの操作は以下のアコーディオンを展開してご覧ください。
<!-- textlint-disable -->
:::details 実行サンプルを見る
<!-- textlint-enable -->
```powershell:PowerShell
New-AzResourceGroupDeployment `
  -Whatif `
  -ResourceGroupName ExampleGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-after.json"
```

[PowerShellのReference](https://learn.microsoft.com/ja-jp/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-8.3.0)

```bash:Azure CLI
az deployment group what-if \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-after.json"
```
[Azure CLIのReference](https://learn.microsoft.com/ja-jp/cli/azure/deployment/group?view=azure-cli-latest#az-deployment-group-what-if)

```bash:REST API
POST https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000001/resourcegroups/my-resource-group/providers/Microsoft.Resources/deployments/my-deployment/whatIf?api-version=2021-04-01

{
  "properties": {
    "templateLink": {
      "uri": "https://example.com/exampleTemplate.json"
    },
    "parameters": {},
    "mode": "Complete"
  }
}
```
[REST APIのReference](https://learn.microsoft.com/en-us/rest/api/resources/deployments/what-if?tabs=HTTP)

:::

What-If操作の結果として以下のような結果が得られ、これからデプロイしようとしているテンプレートが実際のリソースにどのような影響を与えるのかを把握できます。
![](/images/azure-deploy-mode/what-if-result.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/templates/deploy-what-if?tabs=azure-powershell)

## 一部削除されないサービスがある
テンプレートに定義されていない場合でも削除されないリソースがあります。以下のドキュメントの表で`完全モードの削除`が`いいえ`になっている場合は完全モードでも削除されません。
**ただし、`いいえ`になっていても親リソースが削除された場合は削除されます。**
https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/templates/deployment-complete-mode-deletion

## 　リソースがロックされていれば削除されない
Azureではリソースの故意の削除や変更を防ぐために以下の２つの種類の`ロック`を使用して[リソースを保護](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/management/lock-resources?tabs=json#understand-scope-of-locks)できます。
- CanNotDelete：読み取りと変更ができるが、削除ができない
- ReadOnly：削除または更新ができない

**いずれの種類でもロックがかかっているリソースは完全モードを使用したデプロイでも削除されません。** しかし、この挙動を利用してテンプレートには定義せずにリソースを残存させておくのは前述した完全モードの冪等性の思想から、推奨できるものではないと考えます。

## 入れ子のテンプレートは非サポート
完全モードがサポートされるのはルートレベルのテンプレートのみです。 [リンクされたテンプレートまたは入れ子になったテンプレート](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/templates/linked-templates?tabs=azure-powershell)には、増分モードを使用する必要があります。

## コピーループ使用時の注意
コピーループを使用するとプログラミングのfor文のような形で同じリソースをパラメータを変えて反復デプロイできます。完全モードでコピーループが含まれたテンプレートをデプロイする場合、コピーループを解決した後でテンプレートに指定されていないリソースはすべて削除されます。コピーループはリソースが動的に定義される分イメージしづらいので注意しましょう。

## condition利用時のAPIバージョンでの挙動の違い
記事執筆時点で最新版を使用していれば気にすることはありませんが、ワケあって**古いバージョンのAPIを使用し、かつ[condition](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/templates/conditional-resource-deployment)を使用している場合**は注意が必要です。
2019-05-10より前のバージョンを使用する場合、完全モードでもリソースは削除されません。2019-05-10以降では、リソースは削除されます。


# 各モードを使用したデプロイ方法
それぞれのインターフェースでテンプレートデプロイを行う際に`mode`を指定できます。
**規定値はIncremental**なので、完全モードの場合のみCompleteを指定するとOKです。
なお前述の通りAzure Portalでは完全モードを使用はできません。

具体的なデプロイ操作は以下のアコーディオンを展開してご覧ください。
<!-- textlint-disable -->
:::details 実行サンプルを見る
<!-- textlint-enable -->
```powershell:PowerShell
New-AzResourceGroupDeployment `
  -Mode Complete `
  -Name ExampleDeployment `
  -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\storage.json
```
[PowerShellのReference](https://learn.microsoft.com/ja-jp/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-8.3.0)

```bash:Azure CLI
az deployment group create \
  --mode Complete \
  --name ExampleDeployment \
  --resource-group ExampleResourceGroup \
  --template-file storage.json
```
[Azure CLIのReference](https://learn.microsoft.com/ja-jp/cli/azure/deployment/group?view=azure-cli-latest)

```bash:REST API
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/my-management-group-id/providers/Microsoft.Resources/deployments/my-deployment?api-version=2021-04-01

{
  "location": "eastus",
  "properties": {
    "templateLink": {
      "uri": "https://example.com/exampleTemplate.json"
    },
    "parameters": {},
    "mode": "Complete"
  },
  "tags": {
    "tagKey1": "tag-value-1",
    "tagKey2": "tag-value-2"
  }
}
```
[REST APIのReference](https://learn.microsoft.com/ja-jp/rest/api/resources/deployments/create-or-update-at-scope?tabs=HTTP#response)
:::

# 公式ドキュメント
https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/templates/deployment-modes


[^1]: ある操作を1回行っても複数回行っても結果が同じであることをいう概念。
https://ja.wikipedia.org/wiki/%E5%86%AA%E7%AD%89