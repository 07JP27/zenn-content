---
title: "Azure DevOps ServiceのOrganizationが組織のAADへ自由に接続されないように制限する方法"
emoji: "🔒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'azuredevops']
published: true
---

全ては以下のドキュメントにまとまっているのですが、検索しても一発で出てこないことが多いので未来の自分のためにも記しておきます。
https://learn.microsoft.com/ja-jp/azure/devops/organizations/accounts/azure-ad-tenant-policy-restrict-org-creation


# 【前提】Azure DevOps ServiceのOrganizationの仕組み
Azure DevOps Service(以下Azure DevOps)は２種類の認証方法があります。
- Microsoftアカウント
    個人のMSアカウントを使用してログインする方法です。
- Azure Active Directory(以下AAD)
    組織のAADに接続して組織内のユーザーと連携する方法。

本記事は後者のAAD認証の方の話題になります。
AAD認証する場合、Azure DevOpsのOrganizationを作成する際に、AADへ「接続」という処理を明示的/暗黙的に行います。
- AADアカウントでログインした状態でOrganizationを作成すると暗黙的に接続されます。
- Organization作成後に設定画面から明示的に接続・切断も可能です。
![](/images/azure-devops-restrict-organization-connect-to-aad/overview.png)

ちなみにAzure DevOpsは歴史的に見ればVisual Studio Team Foundation Serviceという製品名がリブランディングされたもので、名前の通りAzureではなくVisual Studioブランドの製品でした。なのでAzureのサービスのようで厳密には違うという微妙な立ち位置の製品になっており、ここら辺も紛らわしいポイントのように感じます。

# 何が問題か
既定ではAzure DevOpsにAADの組織アカウントでログインしてOrganizationを作成すると組織のAADにOrganizationが暗黙的に接続できます。
これを許可しておくと自由に社員がOrganizationを作ることができるというメリットの一方、野良Organizationが無数に立ってガバナンス的に良くないという考え方もあると思います。
その場合は、限られたユーザー（管理者）のみがAzure DevOps Organizationを組織のAADに接続できるようにする、という設定をします。

# 接続を制限する方法
ここでまた最初のドキュメント再掲です。
https://learn.microsoft.com/ja-jp/azure/devops/organizations/accounts/azure-ad-tenant-policy-restrict-org-creation

基本的にはドキュメント通りに進めればOKです。
1. 作業者のアカウントをAzure PortalのAADで[Azure DevOps管理者]のロールに割り当てる。
1. Azure Dev OpsのOrganization settingsからAADのメニューで[Restricting organization creation]をオンにする。
1. 例外的にOrganizationを接続できる許可リストを作成する。
1. 制限されたユーザーに対するエラーメッセージを編集する。

# その他に役立ちそうなドキュメント
## すでに接続されているAzure DevOps組織一覧を取得する方法
https://learn.microsoft.com/ja-jp/azure/devops/organizations/accounts/get-list-of-organizations-connected-to-azure-active-directory

## 明示的にAAD接続を行う方法
https://learn.microsoft.com/ja-jp/azure/devops/organizations/accounts/connect-organization-to-azure-ad

## 切断する方法
https://learn.microsoft.com/ja-jp/azure/devops/organizations/accounts/disconnect-organization-from-azure-ad
