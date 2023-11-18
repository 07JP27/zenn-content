---
title: "Azure OpenAIのFunction CallingとJSON Modeの違いと使い所"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# Function CallingとJSON Mode
https://zenn.dev/microsoft/articles/azure-openai-add-function-calling
https://zenn.dev/microsoft/articles/azure-openai-jsom-mode

#　違い
| 比較項目 | Function Calling | JSON Mode |
| --- | --- | --- |
| リクエストの仕方 | ほぼリアルタイム | インデクサーが実行された時点 |
| 返ってくる形 | ユーザーごとのトークンを使うと検索の際に自動フィルターされる | [セキュリティフィルター](https://learn.microsoft.com/ja-jp/azure/search/search-security-trimming-for-azure-search)を使ってインデックスを作成するように構成・実装する必要がある |
| 返ってくるタイミング | ブラックボックス | 全文検索/セマンティック検索/ハイブリッド |


# 想定利用シナリオ

# まとめ
JSON Modeの登場により、JSON形式を保証したかったけどFunction Callingでは返ってくるかわからないし、Functionの定義の手間を考えるとToo muchだったというケースに対応できるようになりました。