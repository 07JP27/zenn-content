---
title: "Promptyでプロンプトセントリックな生成AIアプリ開発をしよう"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Azure', 'OpenAI', 'Prompty', 'VSCode', 'プロンプト']
published: true
publication_name: "microsoft"
---

:::message
本記事のサンプルコードは執筆時点の最新バージョンのAPIとライブラリで検証しています。
特にOpenAI関連のアップデートは頻繁に行われるため、記事閲覧時点の最新バージョンではメソッドやパラメータが変更されている可能性があります。
実際に試す際は最新の公式ドキュメントも併せてご確認ください。
:::

## モチベーション
OpenAIやAzure OpenAIなどのLLMを使ったアプリケーションを開発する際に、プロンプトをどう管理していますか？
コード内に定数として定義したり、データベースに格納したりすることが多いのではないでしょうか。

実際の開発では「まずはプロンプトだけで試して、それでOKならアプリに組み込む」という　ステップを踏むケースが多くあります。
そのような開発フローでその都度プロンプトをコピペを繰り返していると、どれが最新のプロンプトなのかわからなくなりますし、プロンプトだけでなくtemperatureやmax_tokensなどのパラメーターも含めて管理する必要があります。
そのような問題を解決し、**プロンプトと関連パラメーターをアプリケーションの開発ライフサイクルのエンドツーエンドで一元的に管理するためのものがPrompty**です。

## 全体像
Promptyは一言で言うと「**プロンプトや各種パラメーターなどLLM呼び出しに必要な情報をまとめて記述できるファイルフォーマット**」です。
「プロンプトを試す」「プロンプトを改善する」「プロンプトをアプリに組み込む」というLLMを組み込んだアプリケーション開発のフローを各ステップでサポートします。
![](/images/azure-openai-prompty/overview.png)

ここからは各ステップでPromptyがどのように活用されるかを見ていきます。

## Azure AI Studio
プロンプトを試す場合、GUIとして用意されているAzure AI Studioで簡単に試すことができます。
Azure AI StudioでプロンプトやTemperatureなどのパラメーターを試して「これで使えそうだ」となったら、そこからPrompty形式のファイルをエクスポートできます。
![](/images/azure-openai-prompty/ai_studio.png)

## VS Code
VS CodeでのPrompty操作はPrompty拡張機能をインストールすることからはじまります。
https://marketplace.visualstudio.com/items?itemName=ms-toolsai.prompty

VS CodeでPromptyファイルを扱う場合は前述のAzure AI StudioでエクスポートしたファイルをVS Codeで開くか、VS Codeのエクスプローラーで右クリック→「New Prompty」を選択して新規作成する選択肢があります。

### Promptyファイルの構造
まずはベーシックなPromptyファイルを見てみましょう。

```
---
name: ContextPrompt
description: コンテキストを使用したQA回答プロンプト
authors:
  - Junpei Tsuchida
model:
  api: chat
  configuration:
    type: azure_openai
    azure_endpoint: https://xxxxxxxx.openai.azure.com
    azure_deployment: gpt-4o
    api_key: xxxxxxxxxxxxxx
  parameters:
    max_tokens: 3000
sample:
  firstName: ジュンペイ
  context: >
    アルパイン エクスプローラー テントは、プライバシーを確​​保する取り外し可能な仕切り、多数のメッシュ窓と換気のための調節可能な通気口、防水設計を誇ります。アウトドアの必需品を収納できるギアロフトも内蔵されています。つまり、プライバシー、快適さ、利便性が融合した、自然の中心にある第二の家となるのです。
  question: テントについて教えていただけますか?
---

system:
あなたは、人々が情報を見つけるのを助ける AI アシスタントです。アシスタントとして、あなたはマークダウンを使用して短く、簡潔に、そして個性的な方法で質問に答え、適切な絵文字を使用して個人的なセンスを加えることもできます。

# Customer
あなたは、{{firstName}} さんが質問に対する答えを見つけるのを手伝っています。
返信の際にはその人の名前を使用してください。

# Context
次のコンテキストを使用して、よりパーソナライズされた応答を {{firstName}} に提供します: {{context}}

user:
{{question}}
```

Promptyファイルは大きく分けて、「---」によって区切られた部分（本記事では便宜上「設定」と呼びます）と、それ以外の「プロンプト」の２つに分かれています。
すでにAPIやライブラリなどでプロンプトを作成したことがある方は、見慣れたパラメーターが並んでいるため、それほど難しくないと思います。
不明なパラメータがある場合は次の公式のドキュメントに詳細が記載されています。
https://prompty.ai/docs/prompty-file-spec

Promptyファイルの特徴をいくつか挙げます。

### 変数化
プロンプトの部分は通常通り自然言語で記述できますが、Promptyの特徴は変数が定義できる点です。`{{}}`で囲むことによって変数として定義できます。
Prompty ファイル単体として実行する場合は、設定の `sample` セクションに文字通りサンプルの値として定義します。
アプリケーションに組み込む場合は各ライブラリによって変数を渡す方法が用意されています。

### モデルの設定
モデルの設定は上記のサンプルの通り、`model` セクションに記述します。`api` は `chat` か `completion` のどちらかを指定します。`configuration` の `type` には `azure_openai`、`openai`、`maas` のいずれかを指定し、それぞれに対応したエンドポイントやモデルの種類などの設定を記述します。

それ以外の方法としてワークスペースやユーザー設定ごとにモデルを設定しておく方法もあります。設定は `settings.json` に記述します。
`control + shift + P`でコマンドパレットを開いて`setting json`と入力すると`Preferences: Open User Setting(JSON)`が表示されるので選択します。（ワークスペース単位で設定したい場合は`Preferences: Open Workspace Setting(JSON)`を選択します。）
`settings.json` の`prompty.modelConfigurations` には以下のような設定がデフォルトで記述されています。
```json
{
    "name": "default",
    "type": "azure_openai",
    "api_version": "2023-12-01-preview",
    "azure_endpoint": "${env:AZURE_OPENAI_ENDPOINT}",
    "azure_deployment": "",
    "api_key": "${env:AZURE_OPENAI_API_KEY}"
},
{
    "name": "gpt-3.5-turbo",
    "type": "openai",
    "api_key": "${env:OPENAI_API_KEY}",
    "organization": "${env:OPENAI_ORG_ID}",
    "base_url": "${env:OPENAI_BASE_URL}"
}
```
この部分を変更することにより、各ワークスペースごとに異なるモデルを設定できます。
このsettings.jsonに記述された設定を使用する場合はPromptyファイルの`model`セクションに`configuration`を記述する必要はありません。
呼び出すモデル設定の切り替えはVS Codeの右下にあるモデル名をクリックすることで設定されている一覧が表示されて選択できます。
![](/images/azure-openai-prompty/modelswitch1.png)
![](/images/azure-openai-prompty/modelswitch2.png)


### 環境変数の利用
Promptyファイルや`settings.json` の`prompty.modelConfigurations`内で環境変数を利用することもできます。環境変数を利用する場合は`${env:環境変数名}`と記述します。

### JSONファイルの利用
RAGで最終回答を生成するためのプロンプトのように、前の工程からデータがJSON形式で渡ってくるケースがあります。この場合`sample`セクションにJSONをベタ書きするのはPromptyファイルの可読性を損なうため避けたいところです。そのような場合に`${file:ファイルへのパス}`でJSONファイルを参照する方法があります。

また、この仕組みを利用して、プロンプトの変数が複雑になったり、いくつかのサンプルデータを簡単に差し替えてテストしたい場合にも使えます。
たとえば冒頭のPromptyファイルのサンプルは次のようにJSONファイルに切り出せます。
ただし、この状態の場合はアプリケーション組み込みフェーズで、アプリケーションコードから渡す変数もオブジェクト形式で渡す必要がある点に注意しましょう。

```json:data.json
{
    "firstName": "John",
    "context":  "アルパイン エクスプローラー テントは、プライバシーを確​​保する取り外し可能な仕切り、多数のメッシュ窓と換気のための調節可能な通気口、防水設計を誇ります。アウトドアの必需品を収納できるギアロフトも内蔵されています。つまり、プライバシー、快適さ、利便性が融合した、自然の中心にある第二の家となるのです。",
    "question": "テントについて教えていただけますか"
}
```

```
~~~省略~~~
sample:
  data: ${file:data.json}
---

system:
あなたは、人々が情報を見つけるのを助ける AI アシスタントです。アシスタントとして、あなたはマークダウンを使用して短く、簡潔に、そして個性的な方法で質問に答え、適切な絵文字を使用して個人的なセンスを加えることもできます。

# Customer
あなたは、{{data.firstName}} さんが質問に対する答えを見つけるのを手伝っています。
返信の際にはその人の名前を使用してください。

# Context
次のコンテキストを使用して、よりパーソナライズされた応答を {{data.firstName}} に提供します: {{data.context}}

user:
{{data.question}}
```

### 実行
実行はPromptyファイルを開いて、右上の再生マークの「Run」ボタンをクリックするか、`control + shift + P`でコマンドパレットを開いて`Prompty: Run`を選択します。
実行に成功すると`OUTPUT`に結果が表示されます。
![](/images/azure-openai-prompty/result1.png)
また、`OUTPUT`内のドロップダウンから「Promptu Ouput(Verbose)」を選択することでAPIからの生データを確認できます。たとえばusageで使用したトークン数やコンテンツフィルターの適用状況などが確認できます。
![](/images/azure-openai-prompty/result2.png)

#### 実行時の認証
実行の際の認証はAPIキーか、Microsoft Entra ID認証を利用できます。
モデル定義の`configuration`内に`api_key`を指定するとAPIキー認証になり、`api_key`を記述しないとMicrosoft Entra ID認証になります。
Microsoft Entra ID認証を利用する場合は、対象のAzure OpenAIサービスに対して`Cognitive Services OpenAI User`のRBACが割り当たっている必要があります。
割り当て方法は以下の公式ドキュメントを参照してください。

https://learn.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-portal#step-2-open-the-add-role-assignment-page

## アプリケーションへの組み込み
VS Codeでのプロンプトの構築が完了したら、いよいよそれをアプリケーションに組み込んでいきます。
執筆時点で対応しているライブラリやツールはpython、LangChain、Semantic Kernel、Prompt Flowの4つです。
それぞれのコードをゼロからコーディングしても良いですし、`control + shift + P`でコマンドパレットを開いて`Prompty:Add xxx code`を選択することでこれらのコードを生成してくれるオプションもあります。

それぞれPromptyファイルを読み込んで、プロンプトを実行するためのコードを見ていきましょう。本記事ではPromptyに関係する部分のみを抜粋します。実際のコードは依存関係のimportなどが必要です。

### Prompty Core (python)
https://pypi.org/project/prompty/

```python
import prompty

response = prompty.execute("path/to/prompty.prompty")

print(response)
```
prompty.loadメソッドを使うことで、Promptyファイルを読み込んで設定値などを上書きして実行することも可能です。


### LangChain

```python
promptyFilePath = Path(__file__).parent / 'context.prompty'
prompt = create_chat_prompt(promptyFilePath)
promptText = prompt.invoke(
    {
        'firstName': 'ジュンペイ',
        'context': 'アルパイン エクスプローラー テントは、プライバシーを確​​保する取り外し可能な仕切り、多数のメッシュ窓と換気のための調節可能な通気口、防水設計を誇ります。アウトドアの必需品を収納できるギアロフトも内蔵されています。つまり、プライバシー、快適さ、利便性が融合した、自然の中心にある第二の家となるのです。',
        'question': 'テントについて教えていただけますか?'
    }
)

llm = AzureChatOpenAI(
    azure_deployment={デプロイ名},
    openai_api_version={APIバージョン},
)
response = llm.invoke(promptText)
```

### Semantic Kernel
```python
var kernel = Kernel.CreateBuilder()
    .AddAzureOpenAIChatCompletion(
        "{デプロイ名}",
        "{エンドポイント}",
        new AzureCliCredential())
    .Build();

var prompty = kernel.CreateFunctionFromPromptyFile("context.prompty");

var answer = await prompty.InvokeAsync<string>(kernel,
    new KernelArguments
    {
        ["firstName"] = "ジュンペイ",
        ["context"] = "アルパイン エクスプローラー テントは、プライバシーを確​​保する取り外し可能な仕切り、多数のメッシュ窓と換気のための調節可能な通気口、防水設計を誇ります。アウトドアの必需品を収納できるギアロフトも内蔵されています。つまり、プライバシー、快適さ、利便性が融合した、自然の中心にある第二の家となるのです。",
        ["question"] = "テントについて教えていただけますか"
    });
```

LangChainとSemantic Kernelについての詳細は以下の記事でも解説されています。
https://zenn.dev/microsoft/articles/lets-start-prompty


### Prompt Flow
```python
from promptflow.core import Prompty
~~~省略~~~
data_path = os.path.join(pathlib.Path(__file__).parent.resolve(), "./context.prompty")
prompty_obj = Prompty.load(data_path, model=override_model)

result = prompty_obj(firstName = "ジュンペイ", context = "アルパイン エクスプローラー テントは、プライバシーを確​​保する取り外し可能な仕切り、多数のメッシュ窓と換気のための調節可能な通気口、防水設計を誇ります。アウトドアの必需品を収納できるギアロフトも内蔵されています。つまり、プライバシー、快適さ、利便性が融合した、自然の中心にある第二の家となるのです。", question = "テントについて教えていただけますか")
```

## 今後への期待
Prompty自体がまだプレビューということもあり、まだまだ機能追加や改善が期待されます。以下は今回実際に使ってみて感じた現在のPromptyに対する要望です。
### 標準ライブラリへの対応
~~LangChain、Semantic Kernel、Prompt Flowには対応していますが、openai-pythonライブラリやAzure.AI.OpenAIライブラリはまだ未対応です~~
~~これについてはすでにissueが上がっており、対応予定とのことです。~~
~~https://github.com/microsoft/prompty/issues/6~~


→PythonとC#に対応しました🎉
https://github.com/microsoft/prompty/tree/main/runtime/prompty

https://github.com/microsoft/prompty/tree/main/runtime/promptycs



ただ、上記のissueにもコメントがありますが、実際には独自フォーマットのファイルをパースしているだけなので、パース処理を独自実装すればどのようなライブラリやフレームワークでもPromptyを利用できます（もちろん公式サポートしてくれるのが一番ですが・・・）。参考として、以下はPromptFlowのパース実装です。
https://github.com/microsoft/promptflow/blob/1826b3e55e36132281d68e1765f43aa13b6e5241/src/promptflow-core/promptflow/core/_flow.py#L218

### AAD認証のマルチテナント対応
VS Codeでの実行時にMicrosoft Entra ID認証を利用する場合、現状はデフォルトテナントが指定されるようです。テナントIDを指定する方法も見当たらなかったので、デフォルトテナント以外のテナントにデプロイしているモデルにはMicrosoft Entra ID認証が使えないようです。
これについてはissueを作成しました。
https://github.com/microsoft/prompty/issues/29

→対応版がリリースされました🎉詳細はissueをご覧ください。

## まとめ 
プロンプトの管理は単純なシステムプロンプトだけではなくtemperatureやmax_tokensなどのパラメーターを含めた複雑なプロンプトを管理する必要があります。
Promptyはそのような複雑なプロンプトを一元的に管理し、可搬性を向上させるためのフォーマットとして有用です。
今後もPromptyの機能追加や改善が期待されます。

## 参考
https://prompty.ai/

https://build.microsoft.com/en-US/sessions/86e41e8b-1fd2-40fa-a608-6f99a28d4a61

https://learn.microsoft.com/ja-jp/collections/5pq0uompdgje8d?sharingId=ADFFF9D4AD9A0F29&WT_mc.id=aip-114567-cassieb