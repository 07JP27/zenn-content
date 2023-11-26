---
title: "OpenAIのモデルアップデートに備えて評価Notebookを作る"
emoji: "📓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'openai', 'llm']
published: true
publication_name: "microsoft"
---

前回は「OpenAIのモデルアップデートに備えてPrompt Flowでモデルの評価フローを作る」というタイトルでAzure Machine Learningのプロンプトフローを使った評価フローを作りました。
https://zenn.dev/microsoft/articles/llm-testing-using-prompt-flow

そこまで大変な処理はやっていないので、ローカルのJupyter Notebookでも同じことをやってみて、さらにデータをいじくり回してみよう、というのが今回の内容です。
とりあえずNotebookを見たい、という方は[こちら](https://github.com/07JP27/open-ai-test/blob/main/llm-evaluation/evaluation.ipynb)から実行結果も含めて見れます。


# 評価指標
[前回](https://zenn.dev/microsoft/articles/llm-testing-using-prompt-flow)の記事では、モデルの評価指標として、「QnA GPT Similarity Evaluation」というプロンプトフロー組み込みの評価指標を使いました。これは期待値と実測値の類似度を評価してくれるもので、評価自体にもLLMが使われています。
プロンプトフローの場合、この評価指標で使われているプロンプトなどの実装部分がブラックボックスなので、ローカルで評価を行う場合は、自分で評価指標を作る必要があります。
とはいえ、実はプロンプトフローにはQnA Evaluationというテンプレートがあり、これをクローンすると組み込みで使うことができる評価指標の実装がフローとして見ることができます。
![](/images/llm-testing-in-notebook/template.png)
![](/images/llm-testing-in-notebook/template2.png)

プロンプトの部分を変にいじってしまうと、評価結果が変になりかねないので今回はこの信頼と実績(?)の実装をほぼそのまま使います。

# Notebookのポイント
フルのNotebookは実行結果を含みで[こちら](https://github.com/07JP27/open-ai-test/blob/main/llm-evaluation/evaluation.ipynb)から確認できます。
ここではポイントのみ記載します。

## 評価関数
前述の通り、サンプルをほぼそのまま使うのですが、せっかくなので最近使えるようになったJSON ModeでJSONを出力するように変更します。
JSON Modeについてはこちらで説明しています。starというプロパティを持つ`{"star":1}`のようなJSONが出力されてることを期待したコードです。
プロンプトは長いので変数化しています。具体的なプロンプトは[Notebook](https://github.com/07JP27/open-ai-test/blob/main/llm-evaluation/evaluation.ipynb)を見てください。
https://zenn.dev/microsoft/articles/azure-openai-jsom-mode

```python
def gpt_similarity_evaluation(question, answer, ground_truth):
    request_data = {
        "model": evaluation_model['deployment'],
        "temperature":0,
        "messages": [
            {"role": "system", "content": gpt_similarity_evaluation_system_message},
            {"role": "user", "content": gpt_similarity_evaluation_message.format(question=question, ground_truth=ground_truth, answer=answer)},
        ]
    }
    
    # JSON Modeの切り替え
    if evaluation_model['use_json_mode']:
        request_data["response_format"] = {"type": "json_object"}
    response = evaluation_client.chat.completions.create(**request_data)
    try:
        if response.choices and response.choices[0].message:  
            if 'star' in response.choices[0].message.content:
                return json.loads(response.choices[0].message.content)['star']
            else:
                raise ValueError("Error: 'star' not present in the response")
        else:
            raise ValueError("Error: 'choices' or 'message' not present in the response")
    except Exception as e:
        raise ValueError(e)
```

## 対象モデルの定義
対象モデルを配列で定義します。単純にこの配列をfor文で回して、各モデルの評価を行います。
```python
target_models = [
    {
        'label': "gpt35",
        'deployment': '<your-aoai-deployment-name>',
        'apikey': '<your-aoai-api-key>',
        'base': 'https://<your-aoai-resource-name>.openai.azure.com',
        'api_version': "2023-10-01-preview"
    },
    {
        'label': "gpt4",
        'deployment': '<your-aoai-deployment-name>',
        'apikey': '<your-aoai-api-key>',
        'base': 'https://<your-aoai-resource-name>.openai.azure.com',
        'api_version': "2023-10-01-preview"
    }
]
```

またモデル呼び出し時のパラメータはすべての対象モデルで条件を揃えることが望ましいので一括で設定できるようにします。
```python
# 評価実行変数
temperature = 0
top_p=1
max_tokens = 500
evaluation_interval = 10 #sec TPMスロットリング回避のための待機時間
```

## テストデータ
テストデータは[前回](https://zenn.dev/microsoft/articles/llm-testing-using-prompt-flow)用意したものと同じデータを使います。
dataframeで読み込んで対象モデルの数の分、回答と評価結果を入れる列を追加します。
![](/images/llm-testing-in-notebook/1.png)

あとは地道に１レコードずつ「プロンプトを投げては評価関数でスコアリングをして」をテストデータ数×対象モデル数分繰り返して行くだけです。
Azure OpenAIは需要の逼迫でAPIスロットリングがかかることがあるので、APIコールの間隔を開けるために`evaluation_interval`で待機を入れるようにしています。

```python
for target_model in target_models:
    client = AzureOpenAI(
      azure_endpoint = target_model['base'], 
      api_key=target_model['apikey'],  
      api_version=target_model['api_version']
    )
    
    for n_tuple in df.itertuples():
        print(target_model['label']+"-index" + str(n_tuple.Index) + ":Completion in progress" )
        response = client.chat.completions.create(
            model=target_model['deployment'],
            messages=[
                {"role": "system", "content": n_tuple.system},
                {"role": "user", "content": n_tuple.content},
            ],
            temperature = temperature,
            top_p=top_p,
            max_tokens = max_tokens
        )
        answer = response.choices[0].message.content
        df.at[n_tuple.Index, target_model['label']+"-answer"] = answer
        df.at[n_tuple.Index, target_model['label']+"-similarity"] = gpt_similarity_evaluation(n_tuple.content, answer, n_tuple.ground_truth)
        time.sleep(evaluation_interval)
        
print("Done" )
```



# 結果を見てみる
## スコアの分布
今回の評価では期待値に対する実測値の類似度が1~5のスコアで評価されています。まずはこのスコアの分布を見てみましょう。
ヒストグラムとして表示すると以下のようになります。分布の山が右側に行けば行くほど評価が高いことになります。
![](/images/llm-testing-in-notebook/hist.png)

GPT-3.5とGPT-4で差が大きく出ると予想したのですが、今回の評価では大きく差が出ませんでした。若干GPT-4の方が山が右に来ている気もしますが、誤差の範囲かと思います。
この結果から大局的に見れば、今回の評価の内容ではGPT-3.5とGPT-4には差がないと言えそうです。しかし、このヒストグラムからは「本当に各テストデータで同じような類似度を示した」のか「スコアが向上したものと低下したものが同数程度あった」がわかりません。もう少し各テストデータレベルでみてみましょう。

## 範囲での分析
各テストデータでモデルの類似度スコアの最小値と最大値の差の絶対値を求めてみます。
この差があればモデル間で評価が向上または低下していると言え、差が大きいテストデータの件数が少なければ前述の「スコアが向上したものと低下したものが同数程度あった」ことを否定できます。ここではスコアの範囲の中央値の3をしきい値として、その値を超えるテストデータの件数を見てみます。
![](/images/llm-testing-in-notebook/2.png)

その結果4件のテストデータ最小値と最大値の差の絶対値が3以上になっています。テストデータの総数は21件なので、約19%のテストデータでスコアの範囲が大きく変わっていることになります。

また最小値と最大値の差の絶対値が0＝すべての対象モデルでスコアが同じになったテストデータも確認してみます。
![](/images/llm-testing-in-notebook/3.png)

この結果14件のデータがすべてのモデルで同じスコアになっていることがわかります。つまり全体の66%のテストデータで各モデルのスコアが同じになっていることになります。

ここまでをまとめると以下のことが言えます。
- テストデータの19%がスコアの範囲が大きく変わっている。
- テストデータの66%がすべてのモデルで同じスコアになっている。
- それ以外の15%はスコアの範囲が変わっているが、差は小さい。

上記の結果から少なくともいくつかのテストデータは「スコアが向上したものと低下したものが同数程度あった」が、多くは「本当に各テストデータで同じような類似度を示した」と言えそうです。

## 中央値での分析
次に各テストデータの各モデルでの評価の中央値を求めてみます。
これが低い場合は評価対象モデルによらず実測値と期待値に乖離がある傾向を意味し、そもそも期待値が得られるような入力になっていない＝プロンプトエンジニアリングなど入力によって出力を変化させる必要があるのではないか、ということがわかります。
ここでは中央値が「予測された答えが正解と類似していないことがほとんどである。」を意味するスコア２以下をしきい値として、その値を下回るテストデータの件数を見てみます。
![](/images/llm-testing-in-notebook/4.png)

結果として9件、全体の約43%のテストデータで中央値が2以下になっています。
この結果から半分のテストデータはそもそもプロンプトエンジニアリングをもっと頑張る必要があるのではなか、という推測ができます。
（あまりユースケースに基づかないテストデータだったので、たしかにこの結果は予想通りといえば予想通りです・・・。この仮説を実証するにはプロンプトエンジニアリングを真面目にやってみればいいのですが、そこまで気力がありませんでした。）


# Temperatureが大きい場合の工夫
今回の評価ではtemperatureを0にしていますが、temperatureを大きくするとモデルの出力の多様性が上がるため、APIをコールするたびに様々な出力が得られるようになります。
この場合、１つのテストデータを１回だけテストしても、たまたまその時の回答が良かった / 悪かった、ということがおこりえます。それに対する対策としては、Temperatureが高ければ高いほど同じテストデータを複数回テストすることが考えられます。 

# まとめ
前回のプロンプトフローを使用すると非常に簡単に評価フローを作ってそれをチーム内で共有したりできましたが、今回のNotebookでの評価では評価指標の実装や可視化、分析などより柔軟に行うことができます。それぞれ一長一短あると思いますので、目的に応じて使い分けると良いでしょう。