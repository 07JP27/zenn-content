---
title: "Azure OpenAIでStructured Outputsを使う！"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Azure', 'OpenAI']
published: true
publication_name: "microsoft"
---

2024年8月7日に本家OpenAIから発表されたStructured Outputsに続き、9月4日にAzure OpenAI ServiceでもStructured Outputsが利用可能になりました。


## Structured Outputsのモチベーション
OpenAI(ChatGPT)の利用用途はますます広がり、いまやユーザーとのチャットだけではなく、ユーザーの入力から複雑な意図を解析してアプリケーションに連携したり、エージェントと呼ばれるある自律的に思考するAIプログラムを開発することが増えてきています。

そのようなアプリケーションとの連携を考えて自然言語ではなく構造化されたデータを返すことができる機能が必要になります。
これまでAzure OpenAIではJSON Modeと呼ばれる機能や、Function Callingを少しトリッキーな使い方をしてこの構造化を実現していました。
それらの機能については以下の記事で詳しく解説しています。

https://zenn.dev/microsoft/articles/azure-openai-add-function-calling

https://zenn.dev/microsoft/articles/azure-openai-jsom-mode

https://zenn.dev/microsoft/articles/azure-openai-function-calling-json-mode

JSON Modeは返答が構造化された形式で返ってくることは保証されていましたが、そのデータの中身（スキーマ）が100%守られる保証はありませんでした。
特にJSONスキーマをプロンプトとして自然言語で記述する必要があったため、開発者のプロンプトエンジニアリングのスキルに左右されるなど、不安定な部分があったのも事実です。

今回、Structured Outputsの登場により、これらの機能によって返ってくるデータのスキーマが開発者が指定したスキーマと100%一致することが保証されるようになりました。
Structured OutputsはJSON Modeをより確実に、使いやすくした上位互換と言えると思いますが、JSON Modeが不要になるのかは別途改めて考察してみたいと思います。

## Structured Outputsの前提条件
より確実な構造データ生成を行えるようになったStructured Outputsですが、本記事執筆時点では使えるモデルやバージョンが限られています。
- モデル：gpt-4o
- モデルのバージョン：2024-08-06 以上
- APIのバージョン：2024-08-01-preview 以上

## 使ってみる
決まったスキーマを確実に返すようにするためにはスキーマを定義する必要があります。これが結構手間ですが、Pythonを使用している場合pydanticというライブラリを用いてPythonのクラスからJSONスキーマを生成することができます。(便利！)　
RESTでリクエストを送る場合はRESTのBodyにJSONスキーマを記述する必要があります。具体的な方法は[公式ドキュメント](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/structured-outputs?tabs=rest)に記載されています。

ここではキー要素のみを抜き出して紹介します。
実際に動作するコードはGitHubに[Notebook](https://github.com/07JP27/azureopenai-essential-notebooks/blob/fc8b2f21c3f6ef7bc44b9931273b84791a149925/Feature/4_Structured_outputs.ipynb)をアップしていますので、そちらを参照してください。

```python
from openai import AzureOpenAI
from dotenv import load_dotenv
import os
from pydantic import BaseModel

client = AzureOpenAI(
  azure_endpoint = os.environ['BASE'], 
  api_key=os.environ['API_KEY'],  
  api_version="2024-08-01-preview" #2024-08-01-preview以上にする必要あり
)

##構造として出力してほしいスキーマのモデル
class IngredientsToMakeDish(BaseModel):
    dishName: str
    existingIngredients: list[str]
    needToBuyIngredients: list[str]

#呼び出すメソッドが通常のChatCompletionとは異なるので注意
completion = client.beta.chat.completions.parse( 
    model=os.environ['DPLOYMENT'],
    messages=[
        {"role": "system", "content": "Extract the dish name and ingredients that already have and need to buy information."},
        {"role": "user", "content": "今日はぶり大根を作ろうと思います。ブリは家にありますが、ネギと大根がないです。そういえばぶり大根には普通入れないタコもあるので入れてみましょう。"},
    ],
    response_format=IngredientsToMakeDish
)

print(completion.choices[0].message.parsed)
#dishName='ぶり大根' existingIngredients=['ブリ', 'タコ'] needToBuyIngredients=['ネギ', '大根']
```

👇👇👇👇動作するNotebook👇👇👇👇
https://github.com/07JP27/azureopenai-essential-notebooks/blob/fc8b2f21c3f6ef7bc44b9931273b84791a149925/Feature/4_Structured_outputs.ipynb


## スキーマに使用できる型
[公式ドキュメント](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/structured-outputs?tabs=rest#supported-schemas-and-limitations)に記載がありますが、以下の型が使用できます。おおよその方はサポートとされているのではないでしょうか。
- String
- Number
- Boolean
- Integer
- Object
- Array
- Enum
- anyOf

## 【⚠️注意⚠️】サポートされていないキー
各タイプでサポートされていないキーの名称があります。ぱっと見、実際にその型で使うことができるメソッドがリストアップされているように見えるので、予約語と思っておくのが良いでしょう。ソース：[公式ドキュメント](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/structured-outputs?tabs=rest#unsupported-type-specific-keywords)

| 型    | サポート外のキー                                         |  
|---------|-------------------------------------------------------------|  
| String  | minlength<br>maxLength<br>pattern<br>format                 |  
| Number  | minimum<br>maximum<br>multipleOf                            |  
| Objects | patternProperties<br>unevaluatedProperties<br>propertyNames<br>minProperties<br>maxProperties |  
| Arrays  | unevaluatedItems<br>contains<br>minContains<br>maxContains<br>minItems<br>maxItems<br>uniqueItems |  


## 余談１
Structured OutputsはFunction Callingでもリクエストボディに`strict: true`を指定することで利用することができます。**（並列Function Callingは未対応）**
しかしFunction Callingは元来、リクエストボディにJSONスキーマを記述することができるため、見た目上ではStructured Outputsを使うことによる変化は少ない気がします。
おそらく余談2の方の「内部的にStructured OutputsでJSONが生成される仕組みが新しくなった」ということなのではないかと予測しています。

## 余談２
本家OpenAI社のリリースにStructured Outputsが構造化生成を可能にする内部的な解説が記載されています。
Constrained decoding（制約付きデコード）という技術によって次に生成されるトークンをスキーマの入力に制限することで、スキーマに従ったデータを生成することが可能になっているようです。より詳細が知りたい方は本家OpenAI社のリリースを参照してください。

https://openai.com/index/introducing-structured-outputs-in-the-api/



## 参考
https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/structured-outputs?tabs=rest#function-calling-with-structured-outputs

https://openai.com/index/introducing-structured-outputs-in-the-api/