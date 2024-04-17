---
title: "Azure OpenAI よくある質問"
emoji: "❓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Azure", "openai"]
published: true
publication_name: "microsoft"
---

Azure OpenAI利用について、よく聞かれる質問についてまとめてみました。
**この記事は[GitHub](https://github.com/07JP27/zenn-content/blob/main/articles/azure-openai-faq.md)で管理されています。これも追加したらいいんじゃない？という内容はプルリクエストまたは本記事へのコメントでお気軽にお寄せください！**


## 実際にOpenAIのモデルを使わなくても基本料金はかかりますか？
いいえ。基本的には使った分（トークン≒入出力をさせた文量）だけの従量課金制です、使わなければ課金されないし、逆に使った分だけ青天井で課金されます。
「基本的には」というのはファインチューニング（微調整）をしたモデルを使う場合にはホスティング費用が基本料金としてかかります。
https://azure.microsoft.com/ja-jp/pricing/details/cognitive-services/openai-service/

## プロンプトのログを取りたいのですが、標準機能でできますか？
Azure OpenAIだけではプロンプトのログを取る機能はありません。Azure標準機能の「診断設定」で取得できるログにも出力されません。
そのため、Azure OpenAIを呼び出すアプリケーション側でログを取る方法が一般的です。なるべくコーディングをしたくない場合や、APIとして公開するような場合はAzure API Managementを使うとノーコードでプロンプトログの収集構成ができます。
https://zenn.dev/microsoft/articles/azure-openai-nocode-logging

## やっぱりGPT-3.5と比べてGPT-4は精度がいいですか？
精度の定義によりますが、一般的にははい。たとえば複数のタスクにおいて正しく指示理解を行うことを指す指標のMassive Multitask Language Understanding（MMLU）はGPT-3.5が70%に対してGPT-4は86.4%となっています。
https://openai.com/research/gpt-4

## 入力したデータがモデルの再学習に再利用されないって本当ですか？ソースはありますか？
本当です。以下にステートメントが公表されています。
https://learn.microsoft.com/ja-jp/legal/cognitive-services/openai/data-privacy

```
[拙訳]
あなたのプロンプト（入力）と完了（出力）、あなたのEmbeddings、およびあなたのトレーニングデータは：
他のお客様が利用することはできません。
OpenAI が利用することはできません。
OpenAI のモデルを改善するために使用されることはありません。
マイクロソフトやサードパーティの製品やサービスを改善するために使用されることはありません。
お客様のリソースで使用するために、Azure OpenAIモデルを自動的に改善するために使用されることはありません（モデルはステートレスです。）
微調整されたAzure OpenAIモデルは、お客様の使用に限定されます。
```

## 入力や出力データはMicrosoftのデータ領域には保存されないんですか？
いいえ。特に何も構成されない場合、最大30日間は不正利用監視のために限られたメンバーのみが閲覧できる状態でMicrosoftが保存します、
ただし、これは不正利用監視のためであり、前述の通りモデルの再学習のために保存されたデータが使われることはありません。
また、この不正監視のためのデータ保存はオプトアウトを申請することができます。
https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/abuse-monitoring

## 不正監視のためのデータ保存はオプトアウトの申請が通ったことはユーザー側で確認できますか？
はい、できます。Azureポータルで対象のリソースの概要画面から「JSONビュー」をクリックして表示されるJSONの中に以下のプロパティが定義されている場合は不正監視がオプトアウトされています。
```
{ 
    "name":"ContentLogging",
    "value":"false"
}
```
https://learn.microsoft.com/ja-jp/legal/cognitive-services/openai/data-privacy#how-can-a-customer-verify-if-data-storage-for-abuse-monitoring-is-off

## Azure OpenAIに対してEntra ID認証を強制させたいのですが、アクセスキーを無効にする方法はありますか？
あります。`disableLocalAuth`という設定値をfalseにします。ただし現時点ではこの設定はAzureポータルからは変更できません。ARMテンプレートやAzure CLI、REST APIを使って変更する必要があります。
https://learn.microsoft.com/ja-jp/azure/ai-services/disable-local-auth

## コンテンツフィルターがトリガーされて出力が得られなかった場合でも入力トークン数は課金されますか？
基本的には、はい。「基本的には」というのはコンテンツフィルターがどちらでトリガーされたかによって若干変わります。
- ユーザーの入力に対してコンテンツフィルターがトリガーされた場合：入力トークン数は課金される**場合があります。**
- モデルの出力に対してコンテンツフィルターがトリガーされた場合：出力トークン数は課金**されます。**

https://learn.microsoft.com/ja-jp/azure/ai-services/openai/concepts/content-filter?tabs=warning%2Cpython#sample-response-stream-blocked-by-filters

> プロンプトに対してコンテンツ フィルタリングがトリガーされ、応答の一部として "status": 400 が受信されると、サービスによってプロンプトが評価されたときに、この要求に対して料金が発生する場合があります。"finish_reason": "content_filter" で "status":200 が受信されたときにも料金が発生します。

https://stackoverflow.com/questions/77354628/token-usage-of-content-filtered-messages-in-azure-openai-services

## Streamをオンにした時に出力される単位が本家OpenAIと異なるのはなぜですか？
ソースが見つからなかったのですが、おそらくAzure OpenAI独自の「コンテンツフィルター」のためです。
コンテンツフィルターは入力または出力に対して不適切な内容が含まれていないかを判定します。そのため、出力をある程度の単位でまとめて判定する必要があり、その単位ごとのレスポンスになっていると考えられます。
https://learn.microsoft.com/ja-jp/azure/ai-services/openai/concepts/content-filter

Azure OpenAIでも本家OpenAIと同じように出力をしたい場合、まとまって出力された文章を33ミリ秒間隔で１文字ずつ表示する処理を*フロント側で*実装することで実現できます。
公式サンプルの以下の部分でもその実装がされています。
https://github.com/Azure-Samples/azure-search-openai-demo/blob/49a96602a3244c01dcaec64da40dfbc5f9e1e00b/app/frontend/src/pages/chat/Chat.tsx#L77
