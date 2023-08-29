---
title: "BicepでCosmos DBのファイアウォールを設定する"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'bicep', 'cosmosdb']
published: true
publication_name: "microsoft"
---

# やりたいこと
BicepでCosmos DBをデプロイする際にネットワーク設定の以下の２点をオンに設定したいというケースです。この応用で特定のIPアドレスも許可できます。
- パブリック Azure データセンター内からの接続を受け入れる（Accept connections from within public Azure datacenters
）
- Azure Portal からのアクセスを許可する（Allow access from Azure Portal
）
![](/images/azure-cosmosdb-fwrule-bicep/want.png)

# ポイント
ポイントが２点あるので順番に見ていきます。

## ポイント1： Azure Portalではチェックボックスでもデータ構造的にはフラグではない
Azure Portal上ではチェックボックスUIなので、Bicepでも何かのプロパティに`true`を設定すればいいのかなと思いがちですが、そうではありません。
実際には以下のように**それぞれに紐づいた特定のIPアドレスを許可することによって実現します。**

#### Azure Portal からのアクセスを許可する
Azure Portal には複数のIPアドレスが割り当てられています。通常、日本からの利用であれば「その他のすべてのリージョン」のIPアドレス群を許可すれば良いケースがほとんどかと思います。
![](/images/azure-cosmosdb-fwrule-bicep/portal-ip-list.png)

https://learn.microsoft.com/ja-jp/azure/cosmos-db/how-to-configure-firewall#allow-requests-from-the-azure-portal


#### パブリック Azure データセンター内からの接続を受け入れる
Azureリソースからの接続を許可するにはAzure Portalからのアクセス許可と同様に`0.0.0.0`を許可します。
![](/images/azure-cosmosdb-fwrule-bicep/azure-ip.png)

https://learn.microsoft.com/ja-jp/azure/cosmos-db/how-to-configure-firewall#allow-requests-from-global-azure-datacenters-or-other-sources-within-azure

## ポイント2：APIバージョンによって設定するプロパティが違う
ここがこの記事の本題です。上記のドキュメントでは各IPアドレスを`ipRangeFilter`プロパティに設定するように記載があります。
しかし、[`ipRangeFilter`プロパティ](https://learn.microsoft.com/ja-jp/azure/templates/microsoft.documentdb/2020-03-01/databaseaccounts?pivots=deployment-language-bicep#databaseaccountcreateupdatepropertiesordatabaseaccou)はARM APIバージョン`2020-03-01`までしかサポートされていません。
ではどうするのかというと、APIバージョン`2020-04-01`以降では[`ipRules`プロパティ](https://learn.microsoft.com/ja-jp/azure/templates/microsoft.documentdb/databaseaccounts?pivots=deployment-language-bicep#databaseaccountcreateupdatepropertiesordatabaseaccou)を使用します。

``` bicep:2020-03-01まで
properties: {
    consistencyPolicy: {
      defaultConsistencyLevel: 'Session'
    }
    databaseAccountOfferType: 'Standard'
    locations: [
      {
        locationName: location
      }]
    ipRangeFilter: '0.0.0.0,104.42.195.92,40.76.54.131,52.176.6.30,52.169.50.45,52.187.184.26'
  }
```

``` bicep:2020-04-01以降
properties: {
    consistencyPolicy: {
      defaultConsistencyLevel: 'Session'
    }
    databaseAccountOfferType: 'Standard'
    locations: [
      {
        locationName: location
      }]
    publicNetworkAccess: 'Enabled'
    ipRules: [
      {
        ipAddressOrRange: '0.0.0.0'
      }
      {
        ipAddressOrRange: '104.42.195.92'
      }
      {
        ipAddressOrRange: '40.76.54.131'
      }
      {
        ipAddressOrRange: '52.176.6.30'
      }
      {
        ipAddressOrRange: '52.169.50.45'
      }
      {
        ipAddressOrRange: '52.187.184.26'
      }
    ]
  }
```

## デプロイ！
執筆時点で最新バージョンの`2023-04-15`で`ipRules`を使用した以下のようなテンプレートをデプロイします。

``` bicep:main.bicep
param location string = resourceGroup().location
param name string = 'cosmosnwtest'

resource cosmosDbAccount 'Microsoft.DocumentDB/databaseAccounts@2023-04-15' = {
  name: name
  location: location
  kind: 'GlobalDocumentDB'
  properties: {
    consistencyPolicy: {
      defaultConsistencyLevel: 'Session'
    }
    databaseAccountOfferType: 'Standard'
    locations: [
      {
        locationName: location
      }]
    publicNetworkAccess: 'Enabled'
    ipRules: [
      {
        ipAddressOrRange: '0.0.0.0'
      }
      {
        ipAddressOrRange: '104.42.195.92'
      }
      {
        ipAddressOrRange: '40.76.54.131'
      }
      {
        ipAddressOrRange: '52.176.6.30'
      }
      {
        ipAddressOrRange: '52.169.50.45'
      }
      {
        ipAddressOrRange: '52.187.184.26'
      }
    ]
  }
}

```

```
az deployment group create --resource-group cosmostest --template-file main.bicep
```

デプロイしたリソースにAzure Portalでアクセスしてみると想定通りの設定が反映されていることがわかります。
![](/images/azure-cosmosdb-fwrule-bicep/result.png)


# Tips
デプロイしたほぼ全てのリソースは「概要」ページから「JSONビュー」にアクセスできます。
JSONビューでは、そのリソースのAPIバージョンごとのデータ構造が表示できます。Bicepの構造と完全に同じとはいかないまでもAPIバージョンごとのプロパティのデータ構造が垣間見えるので、Bicepの書き方で詰まったときはここを見てみると解決の糸口があるかもしれません。
![](/images/azure-cosmosdb-fwrule-bicep/tips.png)
![](/images/azure-cosmosdb-fwrule-bicep/json-view.png)


# まとめ
今回使用した`ipRules`プロパティを設定するとAzure Portalのチェックボックスに相当する設定項目だけでなく、例えば自宅のIPアドレスなど特定のIPアドレスを許可することも可能ですし、[`virtualNetworkRules`プロパティ](https://learn.microsoft.com/ja-jp/azure/templates/microsoft.documentdb/2020-03-01/databaseaccounts?pivots=deployment-language-bicep#databaseaccountcreateupdatepropertiesordatabaseaccou)で特定の仮想ネットワークからのACL(Access Control List)を設定することも可能です。

Azureサービス自体のアップデートと共に、それを表すデータ構造は変更されます。
しかし基本的に Portalから設定できるプロパティはBicepでも設定が可能です。
仮にプロパティが更新されて変更されても、その項目をマッピングして読み替えるということが今後も必要になる可能性があります。

Azureサービスごと&APIバージョンごとのデータ構造は以下のドキュメントの「リファレンス」セクションに網羅されています。
https://learn.microsoft.com/ja-jp/azure/templates/
