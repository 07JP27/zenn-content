---
title: "Azureのデプロイモードの違いと使い所"
emoji: "🏗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "arm"]
published: false
---

# デプロイモード
Azureでリソースをデプロイする際はAzure PortalやCLIなど、どのようなUIを使用しても、Azure Resource Manager(ARM)と呼ばれる管理サービスを通してデプロイ（変更・削除を含む）が行われます。

![](/images/azure-deploy-mode/arm.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/management/overview)

ARMがAzure上にリソースをデプロイするときのオプションに**デプロイモード**というものがあり、以下の２つのオプションがあります。
- 増分モード（Incremental mode）
- 完全モード（Complete mode）

# 各モードの挙動について
## 増分モード（Incremental mode）
## 完全モード（Complete mode）

# 完全モードは何のためにあるのか？
ざっくり一言で言うと「IaC(Infrastructure as Code)のため」です。
IaCは宣言型の構成ファイルでリソースの「あるべき姿」を宣言し、それをクラウド基盤(Azureの場合はARM)に与えることで宣言されたリソースがデプロイされます。
IaCのための宣言にはAzure固有だとBicepやARMテンプレートが使われ、マルチクラウドだとTerraformが有名どころです。


# 完全モードを使用するときの注意点
## 一部削除されないサービスがある
https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/templates/deployment-complete-mode-deletion
## What-Ifを使う
## コピーループ時の注意
## APIバージョンでの違い
