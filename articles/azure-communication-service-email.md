---
title: "Azure Communication Service Emailを使ってメール配信サービスを作る！"
emoji: "📨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "sendgrid", "azurecommunication"]
published: true
publication_name: "microsoft"
---

# そもそもの話として
Azure VM 上に SMTP サーバーを構築することは非推奨かつサポート外です。
https://learn.microsoft.com/ja-jp/archive/blogs/jpaztech/smtp-block-announcement-november-2017

そのため、例えばAzure上に構築したWebサービスからの通知メールとしてプログラムからメールを送信したい場合、今まではSendGrid等のような、サードパーティーサービス(PaaS)の利用がAzure公式ドキュメントでも案内されていました。


# Azure Communication Service Eメール機能の一般提供
しかし、ついに今回Azure Communication Service Emailが一般提供したよってメール送信機能もファーストパティーサービスを使えるようになりました🎉
https://techcommunity.microsoft.com/t5/azure-communication-services/simpler-faster-azure-communication-services-email-now-generally/ba-p/3788541


# Azure Communication Serviceとメール通信サービスのデプロイ
Azure Communication Service Emailは、１つのサービスですが、実際には以下の3つのリソースが必要になります。

- Azure Communication Service(日本語では「通信サービス」)
- メール通信サービス
- ドメイン

![](/images/azure-communication-service-email/rg.png)
![](/images/azure-communication-service-email/rgd.png)

簡単に言うと、Azure Communication Serviceに対してメール通信サービス(ドメイン)が紐づくイメージです。

## Azure Communication Serviceのデプロイ
Azure Communication Serviceをサクッとデプロイします。ほとんど設定値がないためリソース名とリージョンを指定するくらいです。
![](/images/azure-communication-service-email/acmsdeploy.png)
**ただし、Azure Communication Serviceのリージョンとのちに作成するメール通信サービス(ドメイン)のリージョンは一致している必要があります。**

## メール通信サービスのデプロイ
つづいてメール通信サービス(ドメイン)をデプロイします。ここもほとんど設定項目はありませんが、リージョンだけAzure Communication Serviceと一致するように気を付けてください。
![](/images/azure-communication-service-email/domaindeploy.png)

デプロイが完了したらメール通信サービス(ドメイン)の画面から「ドメインをプロビジョニングする」→「ドメインの追加」→「Azureドメイン」を選択します。
![](/images/azure-communication-service-email/createdomain.png)

しばらくするとAzureドメインのアドレスが払い出されてドメイン一覧に表示されます。

## ドメインの紐づけ
Azure Communication Serviceの画面から「ドメイン」→「ドメイン接続する」を選択します。
![](/images/azure-communication-service-email/adddomain.png)

接続ドメインの選択画面が表示されるのでデプロイしたメール通信サービス(ドメイン)と作成したドメインを選択して「接続」を選択します。前述のリージョンが不一致の場合、ここでドメインが選択できません。
![](/images/azure-communication-service-email/setdomain.png)

これでメールを送信する準備は完了です。

# 使ってみる
プロ開発者向けにREST APIやSDKが提供されていますが、今回はさくっとLogic Appsからメール送信してみます。
例えばファイルがアップロードされたらメールを送るなどのフローがコーディングなしで作れます。

まずはAzure Communication Serviceの「キー」から接続文字列をコピーしておきます。
![](/images/azure-communication-service-email/acmskey.png)

Logic Appsをデプロイしてフローを作ります。「＋」を選択してCommunication Serviceのコネクタを選択してSend emailコネクタを選択します。
![](/images/azure-communication-service-email/connector.png)

最初にコネクタを配置すると接続文字列を入力するように求められるのでさきほどコピーした接続文字列を入力して接続を作成します。
あとは各項目を設定すればOKです。
Fromに作成したドメイン、Toに送信先メールアドレス（複数可能）、Subjectに件名、Bodyにメール本文を入れます。
もちろんLogic Appsのフローで生成された変数も使用できるので動的に本文や宛先を変えることができます。
編集した後は「保存」もお忘れなく。
![](/images/azure-communication-service-email/setaction.png)

設定したトリガーを発生させるか「トリガーの実行」ボタンからフローを実行してみます。
![](/images/azure-communication-service-email/exec.png)

無事メールを受信しました。
![](/images/azure-communication-service-email/sample.png)

# まとめ
めちゃくちゃ簡単にプログラムからメールを送る仕組みを作ることができました。
気になる料金ですが、メールの件数とメール当たりの容量での課金になります。
たとえば1000メールで各メール500KBの場合、約￥42となります。結構安いのではないでしょうか。
![](/images/azure-communication-service-email/price.png)
ぜひ皆様も使ってみてください！！