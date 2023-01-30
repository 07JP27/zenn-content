---
title: "Azure DevOpsのキホンのキ"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'azuredevops']
published: true
publication_name: "microsoft"
---

日々お客様と接する中でよく聞かれる質問やよくある誤解をまとめました。そのため、特に組織(会社)内での利用を想定した内容になっています。
これからAzure DevOpsを使い始めるという方が公式ドキュメントを読み始める前に、本記事をさらっと読んでいただくとドキュメントを読んだ時の理解が深まると思います。
今後も思いついたら追記していきます。

# その１：Azure DevOpsは特定のサービスの名前ではない
Azure DevOpsという名称はスイート製品の名前であり、実際には以下のサービス(製品)がまとまったものになります。そのため、[特定のサービスを使わないといった設定](https://learn.microsoft.com/ja-jp/azure/devops/organizations/settings/set-services?view=azure-devops)も行うことができます。

**Azure Boards**：かんばんとスクラムの方法を使用して、作業、コードの欠陥、問題の計画と追跡をサポートする一連のアジャイル ツールを提供します。
**Azure Repos**：コードのソース管理用の Git リポジトリまたは Team Foundation バージョン管理 (TFVC) を提供します。 
**Azure Pipelines**：アプリケーションの継続的インテグレーションと配信をサポートするビルドサービスとリリースサービスを提供します。
**Azure Test Plans**：手動/探索的テストや継続的なテストなど、アプリをテストするためのいくつかのツールを提供します。
**Azure Artifacts**：チームは、Maven、npm、NuGet などのパッケージをパブリックソースとプライベートソースから共有し、パッケージ共有をパイプラインに統合できるようにします。

何かを検索する時は「Azure DevOps xxxx」と検索するよりも、特定のサービス名で検索した方がより希望にマッチした結果が返されることがあります。

# その２：Azure DevOpsはAzureのサービスであり、Azureのサービスではない
これが個人的に最も勘違いをされやすい部分だと感じています。
Azure DevOpsは歴史的に見れば[Visual Studio Team Foundation Serviceという製品名がリブランディングされたもの](https://learn.microsoft.com/ja-jp/azure/devops/user-guide/about-azure-devops-services-tfs?view=azure-devops#visual-studio-team-services-is-now-azure-devops-services)で、名前の通りAzureではなくVisual Studioブランドの製品でした。なのでAzureのサービスのようで厳密には違うという微妙な立ち位置の製品になっており、紛らわしいポイントのように感じます。
Azure DevOpsの課金はAzureサブスクリプションに対して請求されますが、Azure DevOps内ではAzureの全般で使えるアクセス制御である方式であるRBACが使えなかったり、と完全に統合されているわけでもないし、完全に離れてもいない、という状況です。

# その３：Azureのテナント内に組織が作られるわけではない
「**その１：Azure DevOpsはAzureのサービスであり、Azureのサービスではない**」に関連しますが、Azure DevOpsの組織を作成してもAzure内に何かしらリソースが作成されるわけではありません、Azure とAzure DevOps(の組織)は **「接続」** という依存関係になっています。
![](/images/azure-devops-most-basic-of-basics/overview.png)
https://learn.microsoft.com/ja-jp/azure/devops/organizations/accounts/connect-organization-to-azure-ad?view=azure-devops

まれにAzureの中にAzure DevOpsが配置されているアーキテクチャ図を見ることがありますが、それは厳密には誤りです。
そして既定の設定では組織のAADユーザーであれば、誰でも自身が作った組織を自分が所属するAADに接続することができるようになっています。
詳しくはこちらをご覧ください。
https://zenn.dev/microsoft/articles/azure-devops-restrict-organization-connect-to-aad

# その４：閉域化したいときの選択
Azure DevOpsには2つのオファリングがあります。
- Azure DevOps Services：クラウド版のサービスSaaS
- Azure DevOps Server：オンプレまたは仮想マシン上にインストールできるソフトウェア

閉域化はネットワークインフラを構成することを意味します。SaaSであるAzure DevOps Servicesではインフラのカスタムをサポートしていません。
つまり、**閉域化をしたい場合はAzure DevOps Serverを選択する必要**があります。当たり前のことですが、インフラ部分までお客様が管理する必要があるためServiceに比べて構築・運用コストが高くなる場合がほとんどです。

ServiceとServerの基本的な機能は同じですが、管理面や認証で異なる部分もあります。詳細の比較は以下のドキュメントに記載されています。
https://learn.microsoft.com/ja-jp/azure/devops/user-guide/about-azure-devops-services-tfs?view=azure-devops#fundamental-differences-between-azure-devops-services-and-azure-devops-server

# その５：完全に閉域化(スタンドアローンに)することはできない
「**その４：閉域化したい**」で閉域化できるのはAzure DevOps Server**への**インバウンド方向のみです。
ライセンス認証の通信などが発生するためAzure DevOps Server**からの**通信を完全にインターネットから遮断する(スタンドアローン)ことはサポートされていません。
具体的に疎通を確立する必要があるドメインとポートは以下に記載されてます。
https://learn.microsoft.com/ja-jp/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops&tabs=IP-V4
https://learn.microsoft.com/ja-jp/azure/devops/server/architecture/required-ports?view=azure-devops-2022


# その６：マネージドIDを使える場合と使えない場合
「**その１：Azure DevOpsはAzureのサービスであり、Azureのサービスではない**」に関連しますが、基本的にAzure DevOpsでマネージドIDを使用することはできません。
唯一使用できる条件は次のとおりです。
- セリフホステッドエージェントを使用している
- そのエージェントをAzuer上の仮想マシンなどのコンピューティング上で動かしている

上記を全て満たす場合にそれらのエージェントのリソースに付与されたのマネージドIDを使用することができます。

https://learn.microsoft.com/ja-jp/azure/devops/pipelines/library/connect-to-azure?view=azure-devops#create-an-azure-resource-manager-service-connection-to-a-vm-with-a-managed-service-identity
> マネージド サービス ID を使用するには、Azure VM でセルフホステッド エージェントを使用する必要があります


# その７：課金は組織単位で行われる（まとめることも可能）
Azure DevOpsの課金体系はライセンスによるものですが、その課金は組織単位でAzureのサブスクリプションに請求されます。
https://learn.microsoft.com/ja-jp/azure/devops/organizations/billing/set-up-billing-for-your-organization-vs?view=azure-devops

【**ここ重要**】
規定では複数の組織を作成して、それぞれに同一のAADアカウントがアクセスしている場合、その組織分のライセンス料金がかかります。(2組織にアクセスしている場合は２組織分のライセンス料金が発生する)
**この時、「複数組織の課金」を有効にすることで同一のAADアカウントであれば複数の組織にアクセスしていても1ライセンス分の料金としてまとめることができます。**

https://learn.microsoft.com/ja-jp/azure/devops/organizations/billing/buy-basic-access-add-users?view=azure-devops#pay-for-a-user-once-across-multiple-organizations

しかし、AADと接続している環境はこの設定を問答無用でオンにすればいいかというとそうではありません。
[請求に関する FAQ](https://learn.microsoft.com/ja-jp/azure/devops/organizations/billing/billing-faq?view=azure-devops)の「Q: 複数組織の課金はすべての顧客にとって意味がありますか?」にも記載があるとおりAzure DevOpsには組織ごとに5ユーザー分の無料ライセンスが付与されていますが、**複数組織の課金をオンにした場合、この無料ライセンスが同一AADに接続されている組織で横断して5ユーザーになります。**
そのため、１つの組織にしかアクセスしないユーザーが多い場合はあえて複数組織の課金をオンにしない、という選択肢もありえます。オンにするかどうかは会社の利用スタイルによって決める部分です。