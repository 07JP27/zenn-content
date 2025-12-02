---
title: "Azure Cosmos DBの消費RUを時系列データで確認する。"
emoji: "🌏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "cosmosdb"]
published: true
publication_name: "microsoft"
---


## モチベーション
Azure Cosmos DBでRUをどれくらい使用しているかを把握するのはコスト管理の観点から非常に重要なことです。
特にProvisioned Throughputを使用している場合、RUの上限はワークロードのピークに設定することが多いため、このピークがどのようになっているのか、仮に平準化処理に改修した際に本当に消費RUも平準化されているかを時系列で確認したい場合があります。

## 前提
消費RUに関するデータはログのCDBPartitionKeyRUConsumptionテーブルに入ります。
このテーブルには以下の項目が含まれており、時系列での分析も出来るようになっています。
- TimeGenerated：RUが消費された日時
- RequestCharge：消費されたRUの量

https://learn.microsoft.com/ja-jp/azure/azure-monitor/reference/tables/cdbpartitionkeyruconsumption

このテーブルに対してログを書き込むにはAzure Cosmos DBの診断設定で、Azure Monitorの診断設定を有効かする必要があります。

## 手順
### 1. Azure Cosmos DBでAzure Monitorの診断設定を有効化する
1. Azure Cosmos DBの診断設定を有効化するには、Azure Portalで対象のCosmos DBアカウントを開き、左側のメニューから「診断設定」を選択し、次に「+ 追加」をクリックします。
![](/images/azure-cosmosdb-cunsumption-ru-time-series/diag.png)

2. 診断設定の名前を入力します。
3. 「ログ」セクションで取得する種類を選択します。今回のケースであれば「PartitionKeyRUConsumption」のみ必須です。
4. 送信先で「Log Analytics ワークスペース」を選択し、データを送信するワークスペースを指定します。この時宛先テーブルは「リソース固有」にします。
5. 「保存」をクリックして設定を保存します。

![](/images/azure-cosmosdb-cunsumption-ru-time-series/setting.png)


### 2. Log Analyticsでクエリを実行する
実際にテーブルにデータが入るまでには数分の時間がかかる場合があります。

まずはテーブル全体をクエリして、データが入っていることを確認します。
1. 1で選択した送信先のLog Analyticsワークスペースを開きます。
2. 左側のメニューから「ログ」を選択します。
3. テーブルを展開して「CDBPartitionKeyRUConsumption」の横にある「実行」をクリックします。
![](/images/azure-cosmosdb-cunsumption-ru-time-series/table.png)

4. レコードが表示されればデータは正常に送信されています。次に時系列で見ていきます。

### 3. 時系列でRUの消費量を確認する
1. モード選択画面を「簡易モード」から「KQLモード」に切り替えます。
2. 以下のクエリを入力して実行します。(Your-AccountNameは実際のアカウント名に置き換えてください。)
```kql
CDBPartitionKeyRUConsumption
| where TimeGenerated > ago(1h)
| where AccountName == "Your-AccountName"
| summarize TotalRU = sum(todouble(RequestCharge)) by bin(TimeGenerated, 1s), PartitionKeyRangeId
| order by TimeGenerated asc
| render timechart
```

3. クエリを実行すると、RUの消費量が時系列で表示されます。以下のようなグラフが表示されるはずです。
![](/images/azure-cosmosdb-cunsumption-ru-time-series/graph.png)

## 注意点
上記のグラフの線の色は物理パーティションを表しています。
Cosmos DBのProvisioned Throughputを使用している場合、RUの上限は各物理パーティション毎に分配されます。
つまりProvisioned Throughputが4000RU/sで、物理パーティションが２つ存在した場合それぞれの物理パーティションには2000RU/sずつ割り当てられ、この時のグラフの上限値は2000RU/sとなります。（上記のグラフ場合、物理パーティションはIDが0の１つだけです）

仮に特定の物理パーティションのみのRU消費が常に高い場合、ホットパーティション状態となり、論理パーティションの再配置などを考慮する必要が出てきます。
https://learn.microsoft.com/ja-jp/azure/cosmos-db/nosql/how-to-redistribute-throughput-across-partitions?tabs=nosql&pivots=azure-cli