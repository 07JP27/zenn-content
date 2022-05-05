---
title: "Azure Reposに置かれたARMテンプレートをAzure PipelineでAzureにデプロイする"
emoji: "⚙️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'azuredevops']
published: true
---

Azure（Piplene）によるAzure（Reposからのデプロイ）のためのAzure（の記事）です。
![](/images/deploy-azure-resource-using-devops/outline.png)


## Azure Repos に ARM テンプレートを置く
今回は一連の流れを確認するために、シンプルなストレージを１つデプロイするARMテンプレートを作成してAzure ReposにPushします。
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "storageAccountsName": {
          "defaultValue": "iacteststorage2022",
          "type": "String"
      }
  },
  "variables": {},
  "resources": [
      {
          "type": "Microsoft.Storage/storageAccounts",
          "apiVersion": "2021-06-01",
          "name": "[parameters('storageAccountsName')]",
          "location": "japaneast",
          "sku": {
              "name": "Standard_LRS",
              "tier": "Standard"
          },
          "kind": "StorageV2"
      }
  ]
}
```

![](/images/deploy-azure-resource-using-devops/repos.png)


## Azure Pipeline をセットアップする
Pipelineを新規作成し、コードの場所はARMテンプレートをプッシュしたリポジトリを選択、続いて「Starter pipeline」を選択します
![](/images/deploy-azure-resource-using-devops/setuppipe1.png)
![](/images/deploy-azure-resource-using-devops/setuppipe2.png)

yamlの編集画面が表示されたら画面右側のタスク検索画面に「ARM」と入力し、「ARM template deployment」を選択します。
（タスク検索画面が表示されていない場合は「Show assistant」をクリックして表示できます）
![](/images/deploy-azure-resource-using-devops/tasksearch.png)
セットアップ画面でAzureとの認証等を行いますが、画面通りに操作すると特に迷うことは無いと思います。「Add」をクリックするとyamlが生成されます。
![](/images/deploy-azure-resource-using-devops/setuptask.png)


yamlはこのようになります。
```yaml
trigger:
- main
pool:
  vmImage: ubuntu-latest
steps:
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'xxxxxxxxxxxxxxxxx-xxxxxxxxxxxxx-xxxxxxxxxxxxx'
    subscriptionId: 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx'
    action: 'Create Or Update Resource Group'
    resourceGroupName: 'IaCTest'
    location: 'Japan East'
    templateLocation: 'Linked artifact'
    csmFile: 'template.json'
    deploymentMode: 'Incremental'
```

ここで重要なのは `templateLocation` を「**Linked artifact**」にすることです。これでAzure Reposのファイルを参照できます。
`csmFile`でRepos内でのARMテンプレートまでのパスを記載します。（今回の場合はrootにあるためファイル名のみになります）


## Azure Pipeline を実行
Pipelineの構成が完了したら「Save & Run」ボタンなどからPipelineを実行します。jobが正常に完了すると以下のような画面になります。
![](/images/deploy-azure-resource-using-devops/taskruns.png)

## Azure を確認
ARMテンプレートで指定したリソースグループを見ると正常にデプロイできていることが確認できます。
![](/images/deploy-azure-resource-using-devops/azureresult.png)

## 最後に
今回は簡単な例で行って見ましたが、環境等による出し分けは [ARM テンプレートのパラメーターファイル](https://docs.microsoft.com/ja-jp/azure/azure-resource-manager/templates/parameter-files)や [Azure Pipeline の変数](https://docs.microsoft.com/ja-jp/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch)などを使用することで実現可能です。
また今回は試していませんが、人間は書きやすい [Bicep](https://docs.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/) で書いてPipelineでそれをビルドして生成されたARMテンプレートをデプロイするなども可能です(多分)。

今回使用したARM template deploymentタスクのパラメーターの詳細は以下の公式ドキュメントに詳細な記載があります。
https://docs.microsoft.com/ja-jp/azure/devops/pipelines/tasks/deploy/azure-resource-group-deployment?view=azure-devops