---
title: "Azure OpenAIについてよくある質問"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Azure", "openai"]
published: false
publication_name: "microsoft"
---

Azure OpenAI利用について仕事などでよく聞かれる質問についてまとめてみました。
**この記事は[GitHub](https://github.com/07JP27/zenn-content/blob/main/articles/azure-openai-faq.md)で管理されています。これも追加したらいいんじゃない？という内容はプルリクエストまたは本記事へのコメントでお気軽にお寄せください！**


## 実際にOpenAIのモデルを使わなくても基本料金はかかりますか？
いいえ、基本的には使った分(入出力をさせた文量)だけの従量課金制です、使わなければ課金されないし、逆に使った分だけ青天井で課金されます。
「基本的には」というのはファインチューニング（微調整）をしたモデルを使う場合にはホスティング費用が基本料金としてかかります。

## プロンプトのログを取りたい。
Azure OpenAIだけではプロンプトのログを取る機能はありません。Azure標準機能の「診断設定」で取得できるログにも出力されません。
Azure OpenAIサービスを呼び出すアプリケーション側でログを取る方法が一般的ですが、なるべくコーディングをしたくない場合や、APIとして公開するような場合はAzure API Managementを使うとノーコードでプロンプトログの収集構成ができます。
https://zenn.dev/microsoft/articles/azure-openai-nocode-logging

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


## 入力されたデータはMicrosoftには保存されないんですか？
いいえ。特に何も構成されない場合、最大30日感は不正利用関しのために限られたメンバーのみが閲覧できる状態でMicrosoftが保存します、
ただし、これは不正利用監視のためであり、前述の通りモデルの再学習のために保存されたデータが使われることはありません。
また、この不正監視のためのデータ保存はオプトアウトを申請することができます。
https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/abuse-monitoring

## コンテンツフィルターがトリガーされて出力が得られなかった場合でもトークン数は課金されますか？
おそらく、はい。ただし公式なドキュメントが見つけられませんでした。
https://stackoverflow.com/questions/77354628/token-usage-of-content-filtered-messages-in-azure-openai-services