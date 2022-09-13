---
title: "Azure Static Web Appsの「エンタープライズレベルのエッジ」の効果はいかほど？"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "staticwebapps"]
published: true
---


# はじめに
## Azure Static Web Appsとは
[Azure Static Web Apps](https://azure.microsoft.com/ja-jp/services/app-service/static/)は静的サイトやシングルページアプリケーション(SPA)をホストできるAzureのサービスです。静的アセットは従来のWebサーバーから分離され、世界各地の地理的に分散したポイントから提供されます。 これにより、ファイルがエンドユーザーに物理的に近づくため、ファイルの提供が大幅に高速になります。さらに、APIエンドポイントはサーバーレスアーキテクチャ(Azure Functionsなど)を使用してホストされ、完全なバックエンドサーバーの管理が軽減されます。
また、単純なホスティング環境だけでなくGitHubやAzure DevOpsのCI/CDパイプラインなど開発プロセスも含んだマネージドサービスとして提供されています。
![](/images/azure-static-webapps-enterprise-grade-edge/overview.png)

## 「エンタープライズレベルのエッジ」とは
[エンタープライズレベルのエッジ](https://docs.microsoft.com/ja-jp/azure/static-web-apps/enterprise-edge?tabs=azure-portal)(以下、本機能)はAzure Static Web Appsのオプション機能で[2022年8月31日に一般提供(GA)が開始されました](https://azure.microsoft.com/ja-jp/updates/generally-available-enterprisegrade-edge-for-azure-static-web-apps/)。
具体的には以下のような機能が有効になり、Azure Static Web Appsの性能を文字通りエンタープライズレベルまで引き上げます。

1. 100都市に及ぶ118か所以上のエッジロケーションでのグローバルプレゼンス。
1. エッジでのアセットのキャッシュ。
1. 分散型サービス拒否 (DDoS) 攻撃からの予防的保護。
1. エンドツーエンドのIPv6接続とHTTP/2プロトコルのネイティブサポート。
1. 最適化されたファイル圧縮。

3~5についてはそれらの機能が有効になるということでおおよそ納得できますが、1,2の機能はどれほどの効果があるかがわかりづらいのが現状です。
そこで今回は`1.100 都市に及ぶ 118 か所以上のエッジロケーションでのグローバルプレゼンス。`および`2.エッジでのアセットのキャッシュ`について、本機能を有効にした場合にどの程度ページ読み込み時間を短縮できるのかを確認してみます。[公式ドキュメント](https://docs.microsoft.com/ja-jp/azure/static-web-apps/enterprise-edge?tabs=azure-portal#caching)によるとキャッシュが有効になると以下のようなメリットがあるとされています。
- CDN: できるだけユーザーに物理的に近いエッジロケーションでコンテンツをキャッシュすると、待ち時間が短縮されます。 
- DNS: DNSレコードをキャッシュにより検索が高速化します。
- ブラウザー: ファイルがブラウザーに格納され、同一の要求に対して返されます。(今回はブラウザキャッシュは検証に含めません)

なお本機能の利用には以下のような前提条件や注意点があります。
- サイトにカスタムドメインが設定されていること（Time to Live (TTL) が48時間未満）
- Azure Static Web AppsをStandardプランで利用していること
- 本機能を有効にすると、追加のコストが発生します（アプリ当たり月額$17.52）

## 本機能を有効にする方法
本機能を有効にする方法はとても簡単でAzure Portalで対象のAzure Static Web Appsを選択して`エンタープライズグレードのエッジ`メニューから`エンタープライズレベルのエッジを有効にする`にチェックを入れて`保存`をクリックするだけです。有効化処理には約20～30分かかる場合がありますが、アプリのダウンタイムは発生しません。
![](/images/azure-static-webapps-enterprise-grade-edge/how-to-enable.png)

# 検証
Nuxt.jsで作成したAzure Static Web Appsのサイトに[WebPageTest](https://www.webpagetest.org/)を使用して同一条件で複数の場所からアクセスをしてみます。

## アメリカ/バージニアから
![](/images/azure-static-webapps-enterprise-grade-edge/speedtest/usa.jpg)

## カナダ/トロントから
![](/images/azure-static-webapps-enterprise-grade-edge/speedtest/canada.jpg)

## 日本/東京から
![](/images/azure-static-webapps-enterprise-grade-edge/speedtest/japan.jpg)


# 結果と考察
すべての場所で多かれ少なかれ読み込み速度が早くなっています。
特にタイムラインを見るとDNSの時間が減少しているのでメリットに挙がっているDNSのキャッシュが効いているように見えます。
差が小さい場所については通信帯域などの状況によるものと、**Azure Static Web Appsは元々、ある程度地理的に分散したポイントからコンテンツ配信をするため、メジャーな場所はデフォルトでエッジがあり、本機能を有効にしても元々のエッジが近いため差が小さいのではないか**と考察しています。
いずれにしても本機能を使用することによりパフォーマンスが向上することが確認できたのでグローバルに利用されるアプリや読み込み速度が要求されるアプリには有効と言えそうです。

※本記事の検証結果は本機能の性能を保証するものではありません。