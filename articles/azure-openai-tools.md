---
title: "Azure OpenAIのAPIにtoolsプロパティが追加されたぞ！"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai"]
published: true
publication_name: "microsoft"
---

# toolsプロパティってなに
簡単にいうと現時点ではFunction Callingの上位互換に相当するものです。Function Callingの説明は以下の記事でしています。

https://zenn.dev/microsoft/articles/azure-openai-add-function-calling

**Function Callingの場合、呼び出される機能は１レスポンス１機能でしたが、toolsを使用すると、複数の機能をまとめて呼び出すことができます。**
今のところtoolsにはfunction(機能)しか定義することができません。
ここからは想像ですが、本家OpenAIのAssistant APIのようにCode InterpreterやRetrievalが将来的には使えるのではないかと予想しています。（なので名前もfunctionsから汎用的なtoolsに変更しているのだと想像しています）
半分余談ですが、Code InterpreterはMicrosoft Copilotにリリースされることが[予告されました](https://blogs.microsoft.com/blog/2023/12/05/celebrating-the-first-year-of-copilot-with-significant-new-innovations/)。

# Function Callingはどうなるの？
本家OpenAIではfunctions/function_callプロパティがすでに非推奨になっています。
おそらくそのうちAzure OpenAIも追従することになると思います。つまり「これからはtoolsを使いましょう」となるということですね。ただし後述しますが、Azure OpenAIの場合、今は対応しているモデルが限られているようなので、「今すぐ」というわけではなさそうです。
https://x.com/07JP27/status/1730515000744419520?s=20


https://learn.microsoft.com/ja-jp/azure/ai-services/openai/how-to/function-calling?tabs=python
> API の 2023-12-01-preview バージョンのリリースに伴い、functions および function_call パラメーターは非推奨になりました。 functions に置き換わるのは tools パラメーターです。 function_call に置き換わるのは tool_choice パラメーターです。


# toolsプロパティの使い方
## 前提
前提としておそらく**GPT-3.5とGPT-4のモデルバージョンが1106-previewでないと使えない**と思います。おそらくというのは、~~この記事で紹介しているtoolsプロパティをはじめ、使い方のドキュメントがAPI定義しかないからです。~~

2023/12/12に確認したところ追加されていました。
https://learn.microsoft.com/ja-jp/azure/ai-services/openai/reference#chat-completions

実際に試してみると少なくとも0613ではエラーが発生して使えませんでした。
API定義のtoolsプロパティもたまたま以下の記事を書いているときに見つけました。
https://zenn.dev/microsoft/articles/azure-openai-latest-api-schema

## 使い方
GitHubに[Notebook](https://github.com/07JP27/open-ai-test/blob/main/tools/tools.ipynb)をアップしていますので、フルコードはそちらを参照してください。

はじめにtoolsを定義します。基本的な定義構造はfunction callingのfunctionsの構造と同じですが、toolのtypeプロパティに"function"を指定する必要があります。（この部分からも今後はtoolsの種類を増やしたいという意図が感じ取れますね）
```python
TOOLS = [
    {
        "type":"function",
        "function": {
            "name": "switch_airconditioner",
            "description": "エアコンの電源をスイッチします。",
            "parameters": {
                "type": "object",
                "properties": {
                    "switch": {
                        "type": "boolean",
                        "description": "オンの場合はtrue,オフの場合はfalse"
                    }
                },
                "required": ["switch"]
            }
          }
    },
    {
        "type":"function",
        "function": {
            "name": "switch_lamp",
            "description": "照明のスイッチをオン・オフします。",
            "parameters": {
                "type": "object",
                "properties": {
                    "switch": {
                        "type": "boolean",
                        "description": "オンの場合はtrue,オフの場合はfalse"
                    }
                },
                "required": ["switch"]
            }
          }
    }
]
```

あとは普通にAzureOpenAIクライアントを作って呼び出すだけです。
tool_choiceプロパティはFunction Callingのfunction_callプロパティと同じもので、機能の呼び出しを強制したりできます。

#### 並列機能呼び出しを期待する入力
```python
response = client.chat.completions.create(
    model=deployment,
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "エアコンと電気をつけてください。"}
    ],
    tools=TOOLS,
    tool_choice="auto",
    temperature=0,
)

if response.choices[0].finish_reason=='tool_calls':
    for tool_call in response.choices[0].message.tool_calls:
        print(tool_call.function)
    
#Function(arguments='{"switch": true}', name='switch_airconditioner')
#Function(arguments='{"switch": true}', name='switch_lamp')
```
#### 単一機能呼び出しを期待する入力
```python
response = client.chat.completions.create(
    model=deployment,
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "エアコンを消してください。"}
    ],
    tools=TOOLS,
    tool_choice="auto",
    temperature=0,
)

if response.choices[0].finish_reason=='tool_calls':
    for tool_call in response.choices[0].message.tool_calls:
        print(tool_call.function)

#Function(arguments='{"switch":false}', name='switch_airconditioner')

```

実行結果を含むフルのコードはこちらからどうぞ
https://github.com/07JP27/open-ai-test/blob/main/tools/tools.ipynb

# まとめ
現時点ではfunctionのみの対応とのことで、複数機能の呼び出しが一番うれしいポイントになります。特に今回のサンプルのスマート家電の制御などのように、一つひとつが独立した機能のパラレル制御がしやすくなったのは大きいと思います。
おそらくは今後、toolsで使える機能がfunctionの他にも増えてくると思うので、今後のアップデートに期待です。