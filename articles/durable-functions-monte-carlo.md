---
title: "Azure Functions - Durable Functionsでモンテカルロシミュレーションを実行する"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'durablefunctions']
published: true
publication_name: "microsoft"
---

## Durable Functions
AWS LambdaやAzure Functionsに代表されるFaaSは、サーバーレスアーキテクチャの一つであり、スケール性に優れて多くの処理を並列実行できます。しかし、FaaSはイベントドリブンな処理をするために、1回あたりの処理にはタイムアウトが設けられていることが多く、長時間かかるタスクには向いていません。例えばAzure Functionsを使ってHTTPのWeb APIを作成する場合、リクエストから[230秒以上処理がかかる場合はタイムアウトしてしまいます](https://learn.microsoft.com/ja-jp/azure/azure-functions/functions-scale#timeout)。

FaaSのスケール性や開発容易性（マネージド）を使いつつ、それらの課題を克服するために、AzureではAzure Functionsの拡張機能である**Durable Functions**を利用することで、タスクを並列実行したりチェーンでつなげたりしながら長時間にわたり実行することができます[^1]。

https://learn.microsoft.com/ja-jp/azure/azure-functions/durable/


[^1]: 実際にはDurable Functionsはチェックポイントを使用して状態を保存し、そこから処理を再開することで長時間の実行を可能にしています。また、この動作はアクティビティ関数やスターター関数には適用されません。詳しくは[こちら](https://learn.microsoft.com/ja-jp/azure/azure-functions/durable/durable-functions-orchestrations?tabs=csharp-inproc#next-steps)

### ファンイン・ファンアウトパターン
Durable Functionsには[さまざまな実行パターンが存在](https://learn.microsoft.com/ja-jp/azure/azure-functions/durable/durable-functions-overview?tabs=in-process%2Cnodejs-v3%2Cv1-model&pivots=csharp)しますが、中でもファンイン・ファンアウトパターンは、複数のタスクを並列実行し、その結果を集約するパターンです。例えば、複数のデータに対して同じ処理を並列で実行したり、その結果を集計するような処理に使われます。[^2]

![](/images/durable-functions-monte-carlo/fan-out-fan-in.png)

[^2]: 最近よくある用途として、生成AIのRAGに使うためのデータのチャンク分割処理において、分割した各チャンクを並列でクレンジング処理するような用途に使われています。

## モンテカルロ法
モンテカルロ法は乱数を用いて行うシミュレーション手法です。

### 円周率の計算
モンテカルロ法を学ぶための最も簡単なテーマとして「円周率の計算」があります。
半径1の円の中にランダムにいくつかの点を打ちます。この時に全ての点の数と円の中に入った数の比率は、円の面積と正方形の面積の比率に等しくなると考えることができます。

![](/images/durable-functions-monte-carlo/circle.png)

$$ \frac{\text{円の中に入っている点の数}}{\text{全ての点の数}} = \frac{r^{2}\pi}{4r^{2}} $$

上記の式を$\pi$について解くと円周率は以下の式で表されます。

$$ \pi = 4 \times \frac{\text{円の中に入っている点の数}}{\text{全ての点の数}} $$

なお、ランダムに打たれた点が円の中に入っているかの判定は中心から円周上の点の距離が必ず1になることと、ピタゴラスの定理を利用してxとyの2乗の和が1以下かどうかで判定します。
![](/images/durable-functions-monte-carlo/pythagorean.png)

上記のシミュレーションを十分な数実行すれば、大数の法則によって真の円周率に収束するはずです。

### 計算の流れ
ここで点を打つ回数を「サンプリング」と呼び、サンプリングによって円周率が求められる回数を「イテレーション」と呼びます。
1000サンプリングを10イテレーション行う場合、それぞれ1000個の点から導出された10個の円周率が求められます。
そして最後にそれらを平均することでより平準化した結果を求めることができます。

![](/images/durable-functions-monte-carlo/flow.png)

## Durable Functionsでの実装
ここで話をDurable Functionsに戻します。
Durable Functionsのファンイン・ファンアウトパターンは処理を並列で多重実行するのに適しているため、モンテカルロシミュレーションにも適していると言えます。

Durable Functionsは1回の実行をコントロールするオーケストレーター関数と、そのオーケストレーター関数から呼び出される複数のアクティビティ関数で構成されます。
また、オーケストレーター関数を起動するためのHTTPトリガーやキュートリガーなど、外部要因によって起動されるスターター関数を作成することがほとんどです。
今回は1イテレーションをファンアウトし、1つのアクティビティ関数で1イテレーションを実行するような実装をしています。

![](/images/durable-functions-monte-carlo/function_flow.png)

以下が主要なコードです。
共通クラスなど一部省略している部分がありますが、全体の完全なコードは[GitHub](https://github.com/07JP27/AzFuncMonteCarlo)で公開しています。

```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.AspNetCore.Http;
using Microsoft.DurableTask.Client;
using Microsoft.DurableTask;
using Microsoft.Azure.Functions.Worker.Http;

namespace AzFuncMonteCarlo
{

    public class Durable
    {
        [Function("durable")]
        public static async Task<HttpResponseData> Starter([HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequestData req,[DurableClient] DurableTaskClient client)
        {
            Config? config = await req.ReadFromJsonAsync<Config>();
            if (config is null) return req.CreateResponse(System.Net.HttpStatusCode.BadRequest);

            var instanceId = await client.ScheduleNewOrchestrationInstanceAsync(nameof(Orchestration), config);
            return await client.CreateCheckStatusResponseAsync(req, instanceId);
        }

        [Function(nameof(Orchestration))] 
        public static async Task<Response> Orchestration([OrchestrationTrigger] TaskOrchestrationContext context)
        {
            Config config = context.GetInput<Config>()!;
            var response = new Response();
            var startTime = context.CurrentUtcDateTime;

            List<Task<double>> parallelTasks = new List<Task<double>>();
            for (int i = 0; i < config.TotalIterations; i++)
            {
                Task<double> task = context.CallActivityAsync<double>(nameof(IterationActivity), config.SamplingPerIteration);
                parallelTasks.Add(task);
            }

           var iterations = (await Task.WhenAll(parallelTasks)).ToList();
            response.Iterations = iterations;

            var endTime = context.CurrentUtcDateTime;
            var duration = (endTime - startTime).TotalSeconds;
            response.DurationSecond = duration;
            response.SimulatedMedianValue = response.Iterations.Median();
            response.SimulatedAverageValue = response.Iterations.Average();
            response.SimulatedModeValue = response.Iterations.Mode();

            return response;
        }

        [Function(nameof(IterationActivity))] 
        public static double IterationActivity([ActivityTrigger] double samplingPerIteration)
        {
            var inCircleCount = 0;

            for (int j = 0; j < samplingPerIteration; j++)
            {
                var (x, y) = Utility.GenerateRandomPoint();
                if (Utility.InCircle(x, y))
                {
                    inCircleCount++;
                }
            }

            // 整数除算を避けるために、doubleにキャスト
            var pi = 4 * ((double)inCircleCount / (double)samplingPerIteration);
            return pi;
        }
    }
}
```

このDurable Functionsに以下のようなリクエストを送ることで、モンテカルロシミュレーションを実行することができます。

```json
{
    "TotalIterations": 10000,
    "SamplingPerIteration": 100000
}
```

Durable Functionsでは、長時間かかる処理を実行する際、リクエストの完了までに時間がかかるため、結果をポーリングするためのエンドポイントを簡単に作成できます。このエンドポイントにリクエストを送信することで、処理の進捗を確認できます。
なお、Durable Functionsで長時間の実行が保証されるのはオーケストレーター関数のみです。そのため、スターター関数は処理実行リクエストを受け取った後、処理を開始し、状態確認用のレスポンスを返します。処理はその後、非同期で継続されます。

```json
{
    "id": "1a63ddxxxxxxx220f0d6eb",
    "purgeHistoryDeleteUri": "https://xxxxxxxxxxxxx.azurewebsites.net/runtime/webhooks/durabletask/instances/1a63ddxxxxxxx220f0d6eb?code=xxxxxxxxxxxxx=",
    "sendEventPostUri": "https://xxxxxxxxxxxxx.azurewebsites.net/runtime/webhooks/durabletask/instances/1a63ddxxxxxxx220f0d6eb/raiseEvent/{eventName}?code=xxxxxxxxxxxxx",
    "statusQueryGetUri": "https://xxxxxxxxxxxxx.azurewebsites.net/runtime/webhooks/durabletask/instances/1a63ddxxxxxxx220f0d6eb?code=xxxxxxxxxxxxx",
    "terminatePostUri": "https://xxxxxxxxxxxxx.azurewebsites.net/runtime/webhooks/durabletask/instances/1a63ddxxxxxxx220f0d6eb/terminate?reason={{text}}}&code=xxxxxxxxxxxxx"
}
```

今回の実装の場合、`statusQueryGetUri`にリクエストを送る(ポーリング)ことで処理の進捗を確認することができ、処理が完了すると以下のような結果が取得できます。

```json
{
	"name": "Orchestration",
	"instanceId": "1a63ddxxxxxxx220f0d6eb",
	"runtimeStatus": "Completed",
	"input": {
		"SamplingPerIteration": 100000,
		"TotalIterations": 10000
	},
	"customStatus": null,
	"output": {
		"SimulatedMedianValue": 3.1412,
		"SimulatedAverageValue": 3.141285119999971,
		"SimulatedModeValue": 3.1404,
		"DurationSecond": 315.2634636,
		"Iterations": [
			3.1076,
			...
			3.146
		]
	},
	"createdTime": "2024-11-05T11:41:37Z",
	"lastUpdatedTime": "2024-11-05T11:47:05Z"
}
```

## 実行
円周率を求めるモンテカルロシミュレーションがWeb APIとして作成できたので、実際にJupyter Notebookを使ってDurable Functionsを実行し、その結果を集計してみます。Notebookも[GitHub](https://github.com/07JP27/AzFuncMonteCarlo/blob/main/result_analyze.ipynb)に公開しています。

モンテカルロシミュレーションでは試行回数が多ければ多いほどより正確な値に近似できると考えられますが、実際にはコンピューティングリソースには限りがあるためサンプリング数を無限にすることはできず、ちょうど良いサンプリング数を見つける必要があります。
そこでまずはイテレーションを200回に固定し、サンプリング数を変数としてシミュレーションを実行してみます。円周率は既知の値であるため、補助的に赤の直線で示しています。
![](/images/durable-functions-monte-carlo/sampling.png)

この結果から1イテレーションあたりのサンプリング数は100000回程度であれば（もちろんどれくらい正確に求めるかは目的によりますが）十分な精度で求めることができることが予想されます。

サンプリング数の目処がついたので、次にサンプリング数を100000回に固定し、イテレーション数を変数としてシミュレーションを実行してみます。
![](/images/durable-functions-monte-carlo/iteration.png)

イテレーションが10回程度では収束が見られませんが、100回程度で山ができ始め、10000回程度で正規分布に近づいていることがわかります。

ここまでの実験から、ある程度の精度が期待できる100000サンプリング10000イテレーションで求められた結果は以下の通りです。
![](/images/durable-functions-monte-carlo/final.png)
- 平均値：3.141632191999967
- 中央値：3.14168
- 最頻値： 3.14168
- 処理時間： 195.2828693秒

実用的に使われる円周率が3.14程度までであるとことを考えると、十分実用的な結果を求めることができたと言えます。


## まとめ
Durable Functionsを使うことでこんなことができます、という紹介でした。
ファンイン・ファンアウトパターンを使ってモンテカルロシミュレーションを実行することで、長時間かかる処理を並列で実行することができました。
また、サンプリング数やイテレーション数を変数としてシミュレーションを実行することで、円周率を求める際の精度や収束を確認することができました。

