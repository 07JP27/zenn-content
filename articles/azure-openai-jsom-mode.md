---
title: "Azure OpenAIでJSON Modeを使う！"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai"]
published: true
publication_name: "microsoft"
---

# JSON Modeとは？?
一言で言ってしまうとLLMモデルからの返答がJSON形式で返ってくることが保証されるモードです。
JSON Modeがない場合、次のように喋りすぎてしまったりしてほしい情報がえられないことがあります。
![例えばこんな問題が・・・](/images/azure-openai-jsom-mode/issue1.png)

LLMモデルとやりとりをしている相手が人間であればよしなにJSON部分だけコピーして使うなどの対応ができますが、一般的にJSONでほしい場合＝アプリケーション連携をする場合かと思います。
今まではこのような問題を解決するためにプロンプトエンジニアリングを用いてなるべく所望のデータ形式を得られるようにプロンプトの試行錯誤が繰り返されていました。それでも望んだ形式でない返答が返ってくる可能性があるので、そのような場合に備えてアプリケーション側でリトライ処理などを実装しますが、リトライしたところで上記の問題が100%解決する保証がないので、どこかで諦めないといけなかったりする場合がありました。


## JSON Modeを使えるモデルバージョン
JSON Modeはgpt-35-turbo-1106とgpt-4-1106-previewで使用することができます。Azure OpenAI Serviceにおいては2023年11月17日に利用可能になりました。本家OpenAI側のリリース日(11月06日)からわずか11日後の追従リリースです。
https://techcommunity.microsoft.com/t5/ai-azure-ai-services-blog/azure-openai-service-launches-gpt-4-turbo-and-gpt-3-5-turbo-1106/ba-p/3985962

## どうやって使うの？
JSON Modeの使用方法は非常に単純でChatCompletions APIへのリクエスト時に`response_format={ "type": "json_object" }`プロパティを追加するだけです。

# Azure OpenAI 特有の考慮事項
## リージョン
本家OpenAIと異なりAzure OpenAI Serviceの場合は「リージョン」という概念があり、特定のリージョン（≒国）を指定してモデルをデプロイすることでデータの移動を特定の国内に制御できるなどエンタープライズ要件にも対応できるようになっています。

利用者は特定のリージョンにAzure OpenAI Serviceをデプロイすることで、そのリージョン内のAzure OpenAI Serviceを利用することができますが、このときにデプロイするリージョンによって利用できるモデルが異なります。特に新しめのモデルは利用できるリージョンが限定されることがあるのでデプロイをする前に利用したいモデルはそのリージョンでデプロイすることができるかを確認しておきましょう。

今回のJSON Modeを使用するにはgpt-35-turbo-1106とgpt-4-1106-previewがデプロイできればいいので[リリース記事](https://techcommunity.microsoft.com/t5/ai-azure-ai-services-blog/azure-openai-service-launches-gpt-4-turbo-and-gpt-3-5-turbo-1106/ba-p/3985962)によるとオーストラリア東部、インド南部、アメリカ東部あたりにデプロイするとどちらのモデルも使えます。執筆時点では日本リージョンはまだ未対応です。

## デプロイ
基本的にリージョンを正しくデプロイしていればモデルをデプロイすることは可能ですが、念のためAzure OpenAI Studioのモデル一覧から利用したいモデルがデプロイできるかを確認しておきましょう。利用可能である場合はその画面から「デプロイ」ができます。
![](/images/azure-openai-jsom-mode/model-available.png)

# やってみる
JSON Modeに対応したモデルのデプロイまでできたら本家OpenAIのPython SDKで使ってみましょう。
余談：このSDKは[v1系がリリースされるときにAzure OpenAI Serviceのサポートを削除するという話](https://github.com/openai/openai-python/discussions/631)が一時話題になりましたが、結局サポートは継続されているようです。
https://x.com/07JP27/status/1721469116853485727?s=20

v0.x系をすでにインストールしている場合は[v1.0.0 Migration Guide](https://learn.microsoft.com/ja-jp/azure/ai-services/openai/how-to/migration?tabs=python-new%2Cdalle-fix)に従って`pip install --upgrade openai`を実行しておきます。

なお実際に試したときに使用した実行可能なノートブックは以下のリポジトリで公開しています。
https://github.com/07JP27/open-ai-test/blob/main/json-mode/py.ipynb

## JSON Modeがないとき〜
まずはJSON Modeがないときの挙動を見てみましょう。違いを明確にするため、あえてプロンプトエンジニアリングは雑にしています。
```python
response = client.chat.completions.create(
    model=deployment, # model = "deployment_name".
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "次の文章からJSONを生成してください。地名はlocation、天気はweatherというスキーマを使用してください。¥n===¥n今日の東京の天気は晴れでしょう。"},
    ]
)
print(response.choices[0].message.content)
# 【出力】
# 以下は、提供していただいた文章から生成されたJSONです。
# ```json
# {
#   "location": "東京",
#   "weather": "晴れ"
# }
```

まさに望んだ（予測した）結果が返ってきましたね。

## JSON Modeがあるとき〜
ではプロンプトを全く変えずにJSON Modeを有効にしてみましょう。
```python
response = client.chat.completions.create(
    model=deployment, # model = "deployment_name".
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "次の文章からJSONを生成してください。地名はlocation、天気はweatherというスキーマを使用してください。¥n===¥n今日の東京の天気は晴れでしょう。"},
    ],
    response_format={ "type": "json_object" } # <-- Here!
)
print(response.choices[0].message.content)
# 【出力】
# {
#   "location": "東京",
#   "weather": "晴れ"
# }
```

バッチリJSONだけが返ってきました。

## もうちょっと高度にしてみる
実際のアプリケーションでは上記で見てきたような平のJSONよりも複雑なJSONを扱うケースが多いと思います。JSON Modeの基礎的な能力は確認できたのでもう少し複雑なJSONを生成してみましょう。
```python
response = client.chat.completions.create(
    model=deployment, # model = "deployment_name".
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "次の文章からJSONの配列生成してください。地名はlocation、天気はweatherというスキーマを使用してください。¥n===¥n今日の東京の天気は晴れでしょう。山形県は曇り、北海道は雪まじりの雨となる見込みです。"},
    ],
    response_format={ "type": "json_object" }
)
print(response.choices[0].message.content)
# 【出力】
# {
#   "weatherData": [
#     {
#       "location": "東京",
#       "weather": "晴れ"
#     },
#     {
#       "location": "山形県",
#       "weather": "曇り"
#     },
#     {
#       "location": "北海道",
#       "weather": "雪まじりの雨"
#     }
#   ]
# }
```

すごすぎる・・・ポイントとして「配列で」という日本語をちゃんと理解しているところ、そしてJSONの仕様に従って、指定されていない「weatherData」という配列のキーも文脈から予測して生成してくれているところでしょうか（本来は人間が指定するべきですが）。

# まとめ
Azure OpenAIサービスでのJSOM Modeが使えるようになりました。これによってアプリケーション連携が更にやりやすくなりました。
一方で、すでに利用可能になっていた機能としてFunction Callingという機能が提供されています。こちらもアプリケーション連携を見据えた機能になりますが、次の記事ではJSON ModeとFunction Callingの違いについて見ていきたいと思います。