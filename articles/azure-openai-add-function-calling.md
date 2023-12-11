---
title: "Azure OpenAIでFunction Callingを使う！"
emoji: "📞"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "openai"]
published: true
publication_name: "microsoft"
---

:::message
本記事で紹介する`functions`プロパティは本家OpenAIで非推奨になりました。今後は`tools`プロパティの使用が推奨されています。
今すぐにではないですが、Azure OpenAIもそれに追従する可能性が高く、最新のAPIバージョンでは`tools`プロパティが追加されています。
`tools`プロパティはFunction Callingの上位互換なので、基礎的な知識として本記事で紹介している内容は無駄ではありませんが、今後に備えて以下の`tools`プロパティの使い方も併せてご覧ください。

https://zenn.dev/microsoft/articles/azure-openai-tools

:::

本家OpenAIに続き、Azure OpenAI ServiceでもFunction Callingが利用可能になりました。本記事ではFunction Callingの概要や仕組み、利用方法などをお伝えします。

# なぜFunction Callingが必要なのか
ChatGPTをはじめとするGPTシリーズのリリースから数ヶ月が経過し、市場が徐々にGPTに慣れてきました。それに伴い、単純な「GPTとのやりとり」だけではなく様々なアプリケーションとの連携の需要が増加してきています。本家OpenAIでも[プラグイン](https://openai.com/blog/chatgpt-plugins)と呼ばれる外部のサービスやアプリケーションとChatGPTを連携するような機能もリリースされています。

しかし、**GPTの出力は自然言語であるためアプリケーションとの連携相性が悪いという弱点**があります。例えば以下のような問題が発生しうる可能性がありました。

## レスポンス形式の問題
- アプリで構造データとして扱いたいから「JSONで出力してね」と言っても余計な日本語をつけてきたり
    ![](/images/azure-openai-add-function-calling/issue1.png)
- それっぽいJSONを出力してくれるが、アプリケーション側のスキーマと合致しないJSONだったり
    ![](/images/azure-openai-add-function-calling/issue2.png)

今まではこのような問題を解決するためにプロンプトエンジニアリングを用いてなるべく所望のデータ形式を得られるようにプロンプトの試行錯誤が繰り返されていました。それでも望んだ形式でない返答が返ってくる可能性があるので、そのような場合に備えてアプリケーション側でリトライ処理などを実装しますが、リトライしたところで上記の問題が100%解決する保証がないので、どこかで諦めないといけなかったりする場合がありました。

## 機能呼び出しのタイミング
例えばGPTからの出力を何かしらのアプリケーションに必ず連携するケースであれば問題になりませんが、昨今Copilotという考えかたでユーザーの入力に応じて会話をしながら様々な機能を呼び出すというアプローチが注目されています。この場合、ユーザーの入力に対してGPTが自然な回答を生成することが期待される場合もあれば、途中でアプリケーション側の機能を呼び出す必要がある場合もあります。この切り替えをルールベースのロジックで行うのはなかなか難しいです。


上記のような問題やプロンプトエンジニアリングの手間を解消し、**GPTとアプリケーションを確実に連携できるような仕組みが必要**になってきました。それを実現するための仕組みが Function Calling です。


# Function Callingの動きと仕組み
まずはFunction Callingを使用した場合の全体の動きを見てみましょう。
前章ではフキダシの中がプロンプトと出力になっていましたが、この図ではあくまで「動きの概要」になっていることに注意してご覧ください。
![](/images/azure-openai-add-function-calling/overview.png)
(↑クリックすると拡大できます)

:::message
ちなみに上記の図ではレシピを検索した後に⑤⑥でさらにGPTに対して文章の生成をリクエストしていますが、これは必須ではありません。検索したレシピをそのままアプリケーションの出力として④の直後に機械的に出力するといったことも可能です（例えばレシピ検索の出力がPDFになっていて、それをそのまま返したい場合など）し、さらにそこから別の機能へ連携する、なども可能です。もちろんアプリケーション内で実装している機能だけでなくアプリケーションをハブとして外部のAPIを呼び出すことも可能です。
:::

ポイントは**リクエストの形式が「チャットの入出力」と「使える機能」と「その機能を使う場合に必要なデータ」に別れている**ことです。今までは入力文（プロンプト）の中で３つ全てを与える必要があり、ここが自然言語ゆえの相性の悪さがありましたが、Function Callingのリクエスト形式としてそれらが分離されたことによって、確実な機能の呼び出しとデータの受け渡しができるようになりました。
それではもう少し具体的に見てみましょう。

## Function Callingのエンドポイント
Fcuntion Callingを使うには通常のChatGPT用のエンドポイント(Chat completions)を使用します。ただし、本記事の執筆時点でFunction Callingを使用するにはapi-versionを最新の`2023-07-01-preview`に設定する必要があります。
Function Calling使用の有無に関わらず、Azure OpenAIの場合は本家OpenAIとは異なり開発者各自が[Azure OpenAIのリソースをデプロイし、その中にベースモデルからのデプロイメントを配置する準備](https://learn.microsoft.com/ja-jp/azure/ai-services/openai/how-to/create-resource?pivots=web-portal)が必要になります。そのためエンドポイントは以下のように各環境ごとの変数を含む形なります。
```
POST https://{Azure OpenAIのリソース名}.openai.azure.com/openai/deployments/{デプロイメント名}/chat/completions?api-version=2023-07-01-preview
```

## Function Callingのリクエスト
Function Callingに対応したChatGPTへのリクエストは以下のようになります。
![](/images/azure-openai-add-function-calling/request.png)
`messages`プロパティは従来のChatGPTにも存在していたシステムプロンプト、ユーザーの入力、そしてGPTの出力が含まれている会話履歴です。
Function Callingで肝となるのがアプリケーションが持っている機能をChatGPTに教えるための`functions`プロパティです。`functions`は配列になっていて、複数の機能を保持することができます。`functions`の各要素は`name`と`description`と`parameters`を持っています。
- name
    文字通り、機能の名前になります。ChatGPTが機能を呼び出す必要があると判断した場合はこの`name`が返ってくる(後述のレスポンスのセクションをで詳解)のでこれをキーとして機能を実行します。
- description
    **ChatGPTは`description`に書いてある文章を参考にして入力に対してどの機能を呼び出すか（もしくは呼び出さずに会話として返答するか）を決めています。** なるべく的確に機能の役割を表す必要があり、今まで試行錯誤してきたプロンプトエンジニアリングスキルはここに活きてくることになるでしょう。
- parameters
    機能を実行する際に引数として受け取る必要があるデータとその説明を記載します。ここの説明も機能全体の説明同様に的確に説明する必要があります。また、`parameters`には必須パラメーターを定義することができて、**仮に機能に当てはまるような入力でも必須パラメーターが欠けている場合は機能を呼び出さずにその必須パラメーターを聞き出す形で会話として返答されます。**

`function_call`プロパティは`functions`に与えた機能の`name`の値を設定することによって問答無用でその機能を呼び出すようにChatGPTに指示することができます。ChatGPT側で自律的に機能を呼び出したり、通常の会話を返答する場合は`function_call`に`auto`を設定します。**基本的には`auto`にすることが多いでしょう。**

## Function Callingのレスポンス
前章のリクエストをした場合、ChatGPTからのレスポンスは以下のようになります。
![](/images/azure-openai-add-function-calling/response.png)

まず第一にChatGPTが「アプリケーション側の機能を呼び出そうとしているのか」それとも「通常の会話を返答しているのか」の判断には`choices[0]`>`finish_reason`を判定します。**この値が`function_call`であれば機能呼び出し、`stop`であれば通常の会話を返答しています。**

`finish_reason`が`function_call`（＝機能呼び出し）の場合、通常のChatGPTの返答である`content`ではなく`function_call`というプロパティが追加され、その中に機能の呼び出しに関する情報が入っています。

`finish_reason`が`stop`（＝会話の返答）の場合、従来のChatGPTの返答として扱うことができるので、`content`に返答内容が入っています。

各レスポンスを比較すると次のようになります。(`choices`のみを抜粋)
![](/images/azure-openai-add-function-calling/compare_response.png)


## 呼び出した機能から得られた戻り値を使って返答を生成する
上記「動きの概要図」の⑤〜⑥の部分です。ChatGPTから機能を呼び出すように指示されたアプリケーションは、機能を呼び出してその結果を会話履歴同様に`message`配列に追加します。この時に`role`を`"function"`、`name`を呼び出した機能の`name`、`content`にその機能からの戻り値を入れてリクエストをします。
![](/images/azure-openai-add-function-calling/function-request.png)

その結果、ChatGPTは今までの会話履歴や機能の呼び出し結果を総合した自然言語での回答を返します。例えば上記のようなリクエストを発行した場合、ChatGPTからのレスポンスは以下のようになり、ぶり大根のレシピを含んだ自然な会話が返ってきます。
![](/images/azure-openai-add-function-calling/function-response.png)


# 試してみる
この先の実行に使用したファイル等については以下のGitHubリポジトリにサンプルコードとして置いていますので、ご自由にお使いください。
https://github.com/07JP27/open-ai-test/tree/main/function-calling

## 素のWeb APIで試す
まずはSDKやライブラリを使わずに素のWeb APIで試してみます。テストにはVS Code拡張機能の[REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)を使います。拡張機能を入れることでVS Code上でWeb APIのテストができるツールです。詳しい使い方はこちらの[記事](https://qiita.com/toshi0607/items/c4440d3fbfa72eac840c)を参考にしてください。
[実際に使用したコードはこちらです。](https://github.com/07JP27/open-ai-test/blob/main/function-calling/pure-api.http)


初めにレシピ検索の機能を呼び出すようなリクエストを送ってみます。少しフェイントっぽい入力にしてみましたが、意図通りにレシピ検索が呼び出しレスポンスが返ってきています。
![](/images/azure-openai-add-function-calling/pure1.png)
(↑クリックすると拡大できます)

それに対してレシピ機能から簡単なレシピが返ってきたという想定でレシピ機能からの戻り値を含んだリクエストを送ってみます。100文字程度しかレシピを与えていませんが、それに対してChatGPTが自然な文章に膨らませて生成しています。
![](/images/azure-openai-add-function-calling/pure02.png)
(↑クリックすると拡大できます)

次にレシピ検索の機能を候補にあげつつ、レシピ検索とは関係のない入力のリクエストを送ってみます。Functionsは変わらずに送っていますが、入力から「呼び出す対象の機能は無い」と判断され通常の会話を返答しています。
![](/images/azure-openai-add-function-calling/pure2.png)
(↑クリックすると拡大できます)

最後に多段階で機能の呼び出しに必要なステップを踏むようなリクエストを送ってみます。**レシピ検索のために`food`プロパティを必須としているため、「レシピを調べたいな」だけでは機能呼び出しのレスポンスが返ってきません。**「どの料理のレシピを調べたいですか？」という返答が返ってくるので、それに対して実際の料理名を入力することでレシピ検索が呼び出しレスポンスが返されます。この部分は過去の会話履歴も加味してくれるChatGPTの強みですね。
![](/images/azure-openai-add-function-calling/pure3.png)
(↑クリックすると拡大できます)


## Pythonライブラリで試す
Jupyter Notebook上で試してみます。本家OpenAIのライブラリを使っています。（呼び出せる機能の一覧はコードないで別途定義しています。）
![](/images/azure-openai-add-function-calling/py.png)

[実際に使用したサンプルコードはこちらをご覧ください。](https://github.com/07JP27/open-ai-test/blob/main/function-calling/py.ipynb)

## .NETライブラリで試す
こちらもJupyter Notebook上で試してみます。こちらも簡単なC#であればPythonと同じようにJupyter Notebook上で実行できるので便利ですね。
.NETのライブラリはAzure公式のものを使っています。（呼び出せる機能の一覧はコードないで別途定義しています。）
![](/images/azure-openai-add-function-calling/dotnet.png)

[実際に使用したサンプルコードはこちらをご覧ください。](https://github.com/07JP27/open-ai-test/blob/main/function-calling/dotnet.ipynb)

# その他のTips
## Functionって最大何個まで登録できるの？
GPT-3.5の場合は64個らしいです。（Bing Chatに聞いたら教えてくれました。）
実際に66個のFunctionを登録してリクエストを投げるとそのようなエラーが返ってきました。
![](/images/azure-openai-add-function-calling/tips1.png)


# まとめ
- Function CallingはChatGPTとアプリケーションを確実に連携するための仕組み。
- ChatGPTからのレスポンスで`finish_reason`が`function_call`になった場合はアプリケーション側の機能を呼び出すように指示されている。
- アプリケーション側の機能を呼んだ結果、そのままアプリケーションの出力とすることも、その結果を再度ChatGPTに投げて回答を生成してもらうことも可能。
- プロンプトエンジニアリングのスキルが不要になったわけではない。（機能の説明やパラメーターの説明を書く時やシステムプロンプトでは依然として使う場面もある）

## 責任あるFunction Callingの実装
本記事では主に技術的な面を取り上げましたが、実際のアプリケーション開発では、Function Callingを実装する際に「ユーザーの確認ステップを設けること」など、UXや、セキュリティについての推奨事項がいくつかあります。詳しくは以下のドキュメントを参照ください。
https://learn.microsoft.com/ja-jp/azure/ai-services/openai/how-to/function-calling#using-function-calling-responsibly

# 参考
https://techcommunity.microsoft.com/t5/azure-ai-services-blog/function-calling-is-now-available-in-azure-openai-service/ba-p/3879241

https://learn.microsoft.com/ja-jp/azure/ai-services/openai/how-to/function-calling

https://platform.openai.com/docs/guides/gpt/function-calling

https://github.com/07JP27/open-ai-test