---
title: "Microsoft純正Decisionモデル「Microsoft-Decision-1」が登場"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["microsoft","jev","foundry"]
published: true
publication_name: "microsoft"
---

## 一言で言うと・・・
数週間前にAI界隈を賑わせたJevに相当するモデルをMicrosoftがリリースし、Microsoft Foundryから利用可能になりました。
https://commandline.microsoft.com/microsoft-decision-1-model-foundry/


## はじめに
通常のLLMは出力に自然言語をとりますが、数週間前に登場したJevを始めとする構造化された推論結果を出力するモデル（本記事では便宜上Decisionモデルと呼称）は出力層が軽いなどの理由から構造化されたデータであればLLMよりも速く動作し、かつコストが安いことで注目されていました。
従来はLLMを使って“自然言語で”JSONを吐き出すStructured Outputなどでこのタスクを行っていたので、オーバーヘッドがありました。その出力層を構造化に特化にすることで、推論コストを軽くすればいいという考え方のように見えます。

（上記はざっくり説明なので詳しくは以下をご参照ください）
https://www.itmedia.co.jp/aiplus/article/2609/20/2000001664/


つまり以前投稿した「MAGIシステムをAzure OpenAIで作る」ようなことが高速かつ低コストでできるようになる、という期待ができます。
https://zenn.dev/microsoft/articles/llm-magi-system


## Microsoft-Decision-1
というわけでMicrosoftからもDecisionモデル「Microsoft-Decision-1」が登場しました。

### 価格
インプットは100万トークンあたり$0.042。アウトプットは無料です。
(Jevのインプットは同じく$0.042、OpenAIのDecisions APIは$0.1)
```
Input tokens cost $0.042 USD per million tokens. Output tokens are free.
```

### デプロイ
詳細なデプロイ手順は割愛しますが、Microsoft Foundryのモデルカタログから「Decision」などと検索するとモデルを検索でき、そこからデプロイできます。
![](/images/ms-decision-1/modelcatalog1.png)
![](/images/ms-decision-1/modelcatalog2.png)

モデルカタログを見るとこのモデルは「Qwen3.5-9B」をベースにしているようです。
```
Technical specs
Microsoft-Decision-1 is built on the open-weight  base model .
```


## 使い方
もっとも簡単な使い方としてモデルのデプロイ完了後にMicrosoft Foundryのプレイグラウンドから使用する方法があります。
![](/images/ms-decision-1/usage1.png)

初期画面で表示される内容を日本語訳してみると以下のようなリクエストになり、どこに何を書けば良いのかが大体わかります。


```txt:State
本日、アカウントの支払いが2回失敗しました。今日中に対応をお願いしたいのですが。
```

```json:Decisions JSON
{
  "urgency": {
    "type": "noul",
    "instructions": "お客様は緊急の対応を必要としていますか？"
  },
  "team": {
    "type": "choice",
    "instructions": "どのチームがこのリクエストを担当すべきですか？",
    "criteria": {
      "billing": "支払いおよび請求書に関する問題",
      "technical": "ソフトウェアおよび連携に関する問題"
    }
  },
  "priority": {
    "type": "score",
    "instructions": "このリクエストの優先度はどのくらい高いですか？",
    "criteria": [
      "通常対応、時間的な制約なし",
      "重要、ある程度の時間的な制約あり",
      "緊急、即時の対応が必要"
    ]
  }
}
```

このリクエストをRunすると以下のような結果が返ってきます。
![](/images/ms-decision-1/usage2.png)

ポイントはDecisions JSONで、ここの中でどのような意思判定をしてほしいのか、その判定項目のタイプは何か、ということを指定することでそれに応じたレスポンスが返ってきます。
これらのプロパティを見るとOpenAIのDecision APIとJevのパラメーター名を混在させたような命名規則になっているようです。([出典](https://dev.classmethod.jp/articles/openai-decisions-overview/#:~:text=%E3%81%A8%E3%81%AE%E9%81%95%E3%81%84-,2.1%20%E9%81%95%E3%81%84%E3%81%AE%E4%B8%80%E8%A6%A7,-%E9%A0%85%E7%9B%AE))

判定タイプの一覧は執筆時点で見つけることができなかったのですが、サンプルとして動いたものは以下の通りです。
（見つけ次第記事を更新します。）

|type|種別|
|---|---|
|noul|trueの確率|
|choice|どれかを選択|
|score|100%を選択肢で配分|

## サンプル
というわけでMAGIシステムの記事でも使ったいつものサンプル質問です。
![](/images/ms-decision-1/sample1.png)
~~なんでやねん！他のは豚汁やないかい！~~

通常yes/noであれば上記のようにnoulのほうが簡単ですが、以下のようにchoiceでyes/noを入れることによって、not sureのような選択も入れられたりします。
![](/images/ms-decision-1/sample2.png)
~~なんでやねん！他のは豚汁やないかい！~~


## 付録：素のAPIリクエスト
執筆時点でSDK対応などは確認できていませんが、近いうちの対応が期待されます。
素のWeb APIを呼ぶ場合は以下のリクエストで呼び出せます。
（なおサンプルのためAPI Keyを使用していますが、運用環境ではEntra ID認証で取得したトークンによる認証を推奨します。）

```bash
curl 'https://{Foundry_Resource}.services.ai.azure.com/providers/microsoft/v1/systemone' \
  -H 'Content-Type: application/json' \
  -H 'api-key: YOUR_API_KEY' \
  -d '{"model":"Decision-1","state":"\"芋煮\"の定義は牛肉と醤油と里芋が入っていることが絶対条件である。","questions":{"result":{"type":"choice","instructions":"Stateの質問の答えは？","criteria":{"yes":"はい、そうです。","no":"いいえ、違います。","not_sure":"わかりません"}}}}'
```
