---
title: "Azure OpenAIの返答をAzure Functionを中継してStream(SSE)したい"
emoji: "🏞️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'openai', 'azurefunctions', 'serversentevents', 'streaming']
published: true
publication_name: "microsoft"
---

:::message
本記事では.NETでの実装を紹介しています。Azure Functions Python SDKの仕様により、記事執筆時点ではPythonでStreamを中継することができません。
Streamを中継できるようにするための[SDKのアップデートが予告されている](https://github.com/Azure/azure-functions-python-worker/discussions/1349)ものの、具体的なリリース予定日は未定のようです。
:::

クライアントとOpenAIのエンドポイントの間にAzure Functionsを配置するケースが時折存在します。

- AOAIのキーを隠蔽したい
- ユーザー認証などの処理をしたい
- クライアントに余計なプロパティを設定させたくない
- 複数のAOAIを切り替えて負荷分散をしたい

などなど、理由は様々です。

場合によってはAzure FunctionsだけではなくAPI ManamgenetやApp Serviceを使った方がよい場合もありますが、FaaSのお手軽さを享受するためにもAzure Functionsを使ってみます。通常のChatCompletionAPIへのリクエストであればリクエストとレスポンスの単純な通信で済むのですが、多くの場合Streamを使って生成結果をヌルヌル表示したいことが多いと思います。今回はクライアントとAzure OpenAIの間にAzure Functionsを配置して、Stream(SSE)を有効にした状態でChatCompletionを行う方法を紹介します。

# 実装
ポイントのみ一部抜粋で紹介します。すべてのコードは[こちらのリポジトリ](https://github.com/07JP27/AoaiStreamFunction)にあります。
## Azure Functions
APIの仲介をするのでHTTPトリガーの関数を作成します。
ポイントとしてはつぎの通りです。
- リクエストのHttpContextからレスポンスを取得。
- GetChatCompletionsStreamingAsyncを呼び出すと、OpenAIのAPIからのレスポンスがストリーミングされる。
- そのデータをWriteAsyncで適宜データを書き込み、FlushAsyncでバッファされている出力をクライアントに送信する。

isolatedモードの場合は送信の順番が前後する現象が発生したため暫定処理で遅延を追加しています。（In-procモードの場合は現象が発生しなかったため調査中です。）

```csharp
var response = req.HttpContext.Response;
response.Headers.Add(HeaderNames.ContentType, "text/event-stream");
response.Headers.Add(HeaderNames.CacheControl, CacheControlHeaderValue.NoCacheString);
string dataformat = "data: {0}\r\n\r\n";
await foreach (StreamingChatCompletionsUpdate chatUpdate in await client.GetChatCompletionsStreamingAsync(chatOptions))
{
    log.LogInformation(chatUpdate.ContentUpdate);
    await response.WriteAsync(string.Format(dataformat, chatUpdate.ContentUpdate));
    await response.Body.FlushAsync();
    await Task.Delay(10);
}

await response.WriteAsync(string.Format(dataformat, "[DONE]"));
await response.Body.FlushAsync();
return new EmptyResult();
```

## クライアント
SSEを実装したサーバーはMIMEタイプtext/event-streamで応答が返されます。
この応答は[ReadableStream](https://developer.mozilla.org/ja/docs/Web/API/ReadableStream)にラップされているため、res.body.getReader()でデータを読み取るための[ReadableStreamDefaultReader](https://developer.mozilla.org/ja/docs/Web/API/ReadableStreamDefaultReader)が取得できます。
このReadableStreamDefaultReaderのread()メソッドを使用することでストリームのチャンクへのアクセスを提供するプロミスを取得できます。
この時に返されるdoneがfalse以外のときにはエラーまたはストリームがすでに閉じられていることを示しています。doneがfalseの場合は、valueにはチャンクのデータが含まれているため、それをデコードして表示します。デコードを行うと通常のテキストデータとして操作できるようになるため、不要な文字などを整形して表示します。

```javascript
const res = await fetch("http://localhost:7221/api/chat-stream", {
    headers: {
        "content-type": "application/json",
        "accept":"text/event-stream"
    },
    method: "POST",
    body: JSON.stringify({
        messages:messages
    }),
});

const reader = res.body.getReader();
const decoder = new TextDecoder();
var assistant_message = "";
const read = async () => {
    const { done, value } = await reader.read();
    if (done) {
        return;
    }
    assistant_message += decoder.decode(value).split('data:')[1].trim();
    document.getElementById("assistant-temp-message").innerHTML = assistant_message;
    return read();
};

await read();
reader.releaseLock();
```

左側の黒地の画面がFcuntionのデバッグログ、右側が実際に実装をしたクライアントの画面です。
![](/images/azure-openai-stream-azure-function/demo.gif)

# サンプルコード
実際に動作するサンプルコードをGitHubにアップしています。今回のメインテーマに関係ない以下のような処理は省略しています。実際に使用する際はご注意ください。
- エラー・例外処理
- finish_reasonなどの判定処理

https://github.com/07JP27/AoaiStreamFunction

# 参考
https://ayuina.github.io/ainaba-csa-blog/serversentevents-from-function/