---
title: "Azure Open AIの「Add your data」で出来ること出来ないこと"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai"]
published: false
publication_name: "microsoft"
---

# はじめに
最近リリースされたAzure Open AIの「Add your data」は大きな注目を集めています。
**「Add your data」はAzure固有の機能というわけではなく、Cognitive Searchを使用した検索とそこから回答を得るためのプロンプトエンジニアリングなどの実装をAzureがマネージドサービスとして提供しているものです。**
そのため「Add your data」のような機能を自前で実装することも可能です。
実際、Azureの実装サンプルとして社内ナレッジ検索を自前実装するサンプルが公開されています。
https://github.com/Azure-Samples/azure-search-openai-demo

今回は同じ独自ナレッジの検索機能付きChatGPTという面から「Add your data」と自前実装の違いについて見ていきます。「Add your data」の機能についてまだご存じない方は以下の記事をご一読ください。
https://zenn.dev/microsoft/articles/azure-openai-add-your-data


# 「Add your data」で出来ること出来ないこと
ここから先の情報は本記事の執筆または更新時点の情報です。出来る限り最新情報に追従したいと思いますが、必ずしも最新はないことにご注意ください。

||Add your data|自前実装|
|:--|:--|:--|
|ノーコードでの実現|〇|△|
|閉域化|×|〇|
|プロンプト改善|×|〇|
|セマンティック検索|×|〇|
|日本語の精度向上|△|〇|
|Cognitive Search以外のデータソース|×|〇|

## ノーコードでの実現
## 閉域化
## プロンプト改善
## セマンティック検索
Cogntive Searchにはセマンティック検索という機能があります。これは「セマンティック」の名の通り「意味的に」近しい検索を行うものです。全文検索とは異なり違うワードや文章でも意味的に近しい答えを回答してくれるため、回答精度の向上が期待できます。
https://learn.microsoft.com/ja-jp/azure/search/semantic-search-overview

「Add your data」に使用されているAPIのプロパティにもCognitive Searchのセマンティック検索を有効にするプロパティがありますが、現在は英語のみのサポートとなっています。

Cognitive Searchに対して日本語のセマンティック検索を行うこと自体はできるので、自前実装の場合はセマンティック検索が可能という形になります。

## 日本語の精度向上
## Cognitive Search以外のデータソース


# まとめ
「Add your data」はノーコードでの独自ナレッジ検索の実現が可能ですが、その分機能は限定的です。自前実装の場合は、プロンプト改善やセマンティック検索などの機能を追加することが可能ですが、その分実装コストがかかります。要件に合わせて使い分けることでスピーディーかつ堅牢なシステム構築が可能になります。