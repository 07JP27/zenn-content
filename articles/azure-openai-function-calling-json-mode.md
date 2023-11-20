---
title: "Azure OpenAIのFunction CallingとJSON Modeの違いと使いどころ"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai"]
published: true
publication_name: "microsoft"
---

ChatGPTが世の中にリリースされてから早１年が経とうとしています。この１年の間に様々な驚くべきアップデート(一番驚いたのがCEO解任というのは置いておいて)が提供されています。
特にアプリケーションにLLMモデルを組み込むという観点から見ると6月13日に本家OpenAIのモデルにFunction callingが実装され、開発者が歓喜したのは記憶に新しいのではないでしょうか。
この度11月6日のアップデートでJSOM Modeというものが追加されました。
どちらもアプリケーション組み込みのための機能ですが、それぞれはどう違い、どのような使い方が想定されているのでしょうか。今回はその違いと使いどころについてまとめてみました。

# Function CallingとJSON Mode
それぞれの基本的な使い方については別途記事にしているのでそちらを参照してください。
本記事では違いについてのみ記載します。

## Function Calling
https://zenn.dev/microsoft/articles/azure-openai-add-function-calling
## JSON Mode
https://zenn.dev/microsoft/articles/azure-openai-jsom-mode

# 違い
JSON Modeが登場するまで、LLMモデルから確実にJSONを取得するために「機能呼び出し時の引数がJSONで返ってくる」という性質を利用してFunction Callingを利用するケースがありました。つまり実際には存在しない機能(Function)を定義して、その機能を呼び出すようなプロンプトを入力することで機能の実行をトリガーして返ってきた引数のJSONだけをデータとして取得・利用するという方法です。
上記のような事情をご存じの方は、Function Callingでカバーできていた機能をなぜ後発で実装したのか、Function Callingではだめなのかという疑問が生まれるかもしれません。そのような観点からそれぞれの違いを表にまとめてみました。

| 比較項目 | Function Calling | JSON Mode |
| --- | --- | --- |
| リクエストの仕方 | functionsプロパティに、LLMモデルが使用することができる機能の名前や説明、それを呼び出すために必要な引数の形（JSON）を定義 | API呼び出し時に`response_format={ "type": "json_object" }`を付与する（メッセージには「json」を含む必要あり） |
| 返ってくる形 | 定義した機能の中から呼び出すべき機能名とその引数となる値（JSON文字列） | JSON文字列 |
| 返ってくるタイミング | LLMがユーザーの入力に基づいて、定義されている機能を呼ぶべきだと判断したとき | `response_format={ "type": "json_object" }`をつけて呼び出している限り常に |
| JSONスキーマの指定方法 | functionsプロパティ内で定義 | メッセージ内(プロンプトエンジニアリング)で定義 |

このような違いからFunctions CallingはJSON Modeの上位互換という見方もできるかもしれませんが、**JSON Mode特有の強みとして常にJSONでの戻りを強制できる、という点があります。**


## 想定利用シナリオ
上記の違いから考えると、今後は以下のような使い分けをするとよさそうです。

### Function Calling
いわゆるCopilot（ReActエージェント）としていつ呼び出されるか不定な様々な外部機能を有したアプリケーションを作るときに使用します。定義された機能を呼び出すかどうかということもLLMモデルが判断します。

### JSON Mode
例えばRAGアーキテクチャでユーザーの入力から検索クエリを作成するなど、シーケンシャルな処理の一部としてLLMを使用する場合に使用します。JSON Modeを有効にしてリクエストすれば必ずJSON文字列が返ってくることが保証されます。ただしスキーマの指定などはFunction Callingと異なり通常のメッセージの中で行う必要があるためプロンプトエンジニアリングは必要です。

# まとめ
JSON Modeの登場により、JSON形式を保証したかったけどFunction Callingでは機能呼び出しのレスポンスが返ってくるかわからないし、Functionの定義の手間を考えるとToo muchだったというケースに対応できるようになりました。ますますAzure OpenAI Serviceのアプリケーション組み込みが楽になりそうです。