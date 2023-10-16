---
title: "Azure API Centerとは？"
emoji: "🗂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure']
published: true
publication_name: "microsoft"
---

本記事ではAzure API Centerについて記事更新日時点の情報に基づいてまとめています。
Azure API Centerは記事執筆時点でパブリックプレビューとなった新しいサービスで、機能が限られています（後述するデモ動画では「Very limited functionality」と表現されている）。そのため、読者の方が本記事を読んでいる時点での情報とは異なる可能性があります。ご注意ください。



# Azure API Centerとは？
一言で説明するとAzure API Centerとは企業内のAPIをまとめて**カタログ化**できるサービスです。
API Managementには開発者ポータルというAPIのスキーマなどを確認できる同じような機能がありますが、1つのAPI Managementに対して1つの開発者ポータルが存在する(1 to 1)のに対して、API Centerは複数のAPIソリューションを1つにまとめる(many to 1)ことができます。
## ポイント
- 複数のAPIを集約して企業内のAPIカタログを定義できる
- APIはAzureだけではなくてapigeeやMuleSoftなどサードパーティーも集約可能
- AzureのRBACによって細かい権限制御（閲覧だけ許可など）が可能
- 少なくとも記事執筆時点では本当にAPIの情報を見るだけで、API CenterからAPIを呼び出したりなどはできない


# Azure API Centerのエコシステム
Azure API Centerの想定される使用方法は以下のようになっています。
1. API提供者がAPIを開発してデプロイする
1. API提供者がデプロイしたAPIの情報をAPI Centerに登録する
1. API利用者がAPI Centerのカタログを確認して呼び出したいAPIについて確認する
1. API利用者が実際のAPIを呼び出す
![](/images/waht-is-api-center/ecosystem.png)


# Azure API Centerの構成コンポーネント
Azure API Centerの中にはいくつかのコンポーネントが存在し、それぞれ以下の図のような関係になっています。
![](/images/waht-is-api-center/components.png)

## 各コンポーネント(詳しい説明)
### Metadata schema
メタデータスキーマはAPIs、Deployments、Environmentsのそれぞれのコンポーネントにカスタムのメタデータを付与できます。Azureの「タグ」のようなものだと考えていただければいいと思います。
例えばコストセンターを作成してAPIに紐づけるといった用途で使われます。タグと比較して様々なデータ型（配列やBoolなど）を定義できるのが特徴です。
![](/images/waht-is-api-center/metadata.png)

JSONの入れ子のような複雑なObject型も定義できるのでとても柔軟に定義できます。
![](/images/waht-is-api-center/metadata_object.png)

メタデータはそれぞれAPI、Deployments、Environmentsのどのコンポーネントに設定できるようにするかアサインできます。この設定を上手に使うことで各コンポーネントの管理をしやすくなります。
- Not applicable：そのコンポーネントには設定できない
- Optional：そのコンポーネントには任意で設定できる
- Required：そのコンポーネントには必須で設定する必要がある
![](/images/waht-is-api-center/metadata_assign.png)

設定したメタデータは各コンポーネントの検索条件として使用できます。
![](/images/waht-is-api-center/metadata_search.png)

### Environments
実際のAPIのホスティング情報を定義するコンポーネントです。APIのタイプ(サービス)とそのエンドポイントを構成し、API Center内でのAPIホスティング環境を表現します。
選択できるAPI Typeは以下の通りで様々なサードパーティAPIも選択できることがわかります。
![](/images/waht-is-api-center/environment_type.png)

### APIs
#### API
Version - DefinitionsとDeploymentsをまとめる単位です。
この単位が複数できることで単一のAPIをカタログに集約できるようになります。

#### Version - Definitions
APIのスキーマ定義を行います。APIがどこでホストされているか、はここでは考慮しません。
そのようにスキーマとホスト環境(Environments)を分離することで、DevelopmentとProductionでスキーマが同じでホスト先が異なる場合でも同じスキーマを使いまわしたりすることができます。
スキーマの対応形式も様々あり、例えばAPI Managementを使用している場合はAPI ManagementからOpen API形式でAPI定義をエクスポートして、API CenterのDefinitionsに読み込ませるという運用になります。
![](/images/waht-is-api-center/schema_type.png)


#### Deployments
最終的にAPIのスキーマ定義とEnvironmentsを紐づけてAPIを定義します。
ここが実際にAPIスキーマとエンドポイントの組み合わてAPIの呼び出し方法を確認できるカタログになります。
![](/images/waht-is-api-center/deployment.png)

# デモ動画
以下の動画で2023年5月に開催されたMicrosoft Build 2023時点でのデモ動画を見ることができます。
https://youtu.be/BNH5kq7Ose0?si=oC8E7gHAr0y1ckgd&t=1098

# API CenterとAPI Managementの関係性
上記のデモ動画ではAPI Centerは「It's not going to be part of Azure API Management.(API Managementの一部にはならない)」としています。
しかし[パブリックプレビューのアナウンス](https://azure.microsoft.com/ja-jp/updates/ungated-public-preview-azure-api-center/)では「Azure API Center is a new Azure service that is part of the Azure API Management platform.（Azure API Centerは、Azure API Managementプラットフォームの一部である新しいAzureサービスである。）」とされており、紛らわしいです。

少なくとも、API CenterはAPI Managementとは別にデプロイするサービスなので一機能ではないということは言えます。おそらく複数のAPI Managementをカタログ化できると言う意味でAPI Managementを使ったエコシステムの一部として使用することができる、という意味合いではないかと想像しています。

# 公式ドキュメント
https://learn.microsoft.com/ja-jp/azure/api-center/overview

https://techcommunity.microsoft.com/t5/azure-integration-services-blog/introducing-azure-api-center-for-centralized-api-discovery-and/ba-p/3827403