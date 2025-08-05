---
title: "MAGIシステムをAzure OpenAIで作る"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aoai", "llm", "azure", "openai"]
published: true
publication_name: "microsoft"
---


## LLMにおける意思決定の課題
近年ではLLMの精度も向上し、簡単なタスクであれば単純に推論をするだけでそれなりの出力が得られるようになってきました。
しかし、実務レベルの意思決定の場面ではいわゆる「ハルシネーション」が発生することもあります。
また、ハルシネーションのような確定事項を誤って判断するだけでなく、「料理のレシピ」などのようにそもそも答えが決定的でない場合もあります。

しかし、ここで重要なのはLLMだから間違うということではなく、人間も同じように間違える可能性があるということです。
では人間が間違わないためにはどうすればよいか？ということですが、複数の人数で多角的に議論をし、合議するという手段が一般的に取られています。
例えば製品の出荷判定会議やソフトウエア設計レビュー会、国会などさまざまなところで合議制が取られます。

もともとLLMにも使われている機械学習の一種であるニューラルネットワークは人間の脳回路を抽象化して作られたものというところに着想し、意思決定のプロセスも人間の合議制を模して作ることができるのではないかと考えました。（調べてないですが、おそらくN番煎じネタです）


## MAGIシステムとは
MAGIシステムは新世紀エヴァンゲリオンに登場する人工知能システムで、複数のAIが合議して意思決定を行う仕組みです。
このシステムでは、複数のAIがそれぞれ独立して推論を行い、その結果を集約して最終的な意思決定を行います。
このような合議制のアプローチにより、個々のAIのハルシネーションのリスクを低減し、より信頼性の高い結果を得ることができます。

おそらくMAGIシステムにも元ネタがあり、新約聖書に登場するイエスの誕生時にやってきてこれを拝んだとされる[東方の三賢者](https://ja.wikipedia.org/wiki/%E6%9D%B1%E6%96%B9%E3%81%AE%E4%B8%89%E5%8D%9A%E5%A3%AB)を指しているのではないかと思います。（賢者のラテン語が「magi」）

同じように映画「[マイノリティ・リポート](https://ja.wikipedia.org/wiki/%E3%83%9E%E3%82%A4%E3%83%8E%E3%83%AA%E3%83%86%E3%82%A3%E3%83%BB%E3%83%AA%E3%83%9D%E3%83%BC%E3%83%88)」でも3人の予言者の合議により、2人以上が同意した場合に犯罪を事前に予知して取り締まるというシステムが描かれています。

## MAGIライブラリを作ろう
議題を投げ込めば勝手に合議してくれるようなライブラリを作っておけば便利なのでは？と思ったのでN番煎じで作ってみました。
いつも通り以下にクラスライブラリとそれを使うサンプルUIのコードを公開しました。

https://github.com/07JP27/MagiSystem

フルコードはそちらを参照していただくとして、ここでは重要な部分だけを抜粋して説明します。

まずは複数の賢者を`Sage`というクラス名で定義しており、インスタンス作成時にパーソナリティを指定できるようになっています。
そして指定された議題に対して投票を行う`VoteAsync`メソッドを持っています。
```csharp
public class Sage
{
    private static readonly ChatOptions _voteChatOption = new()
    {
        ResponseFormat = ChatResponseFormat.ForJsonSchema(AIJsonUtilities.CreateJsonSchema(typeof(SageResponse))),
    };
    private readonly string _systemPrompt;
    private readonly IChatClient _aiChatClient;
    public Sage(string personality, IChatClient aiChatClient)
    {
        _systemPrompt = $"""
        あなたは合議制における投票権を持つ一人の賢者です。与えられた議題に対して投票を行うことが求められます。
        あなたのパーソナリティは「{personality}」です。必ずこのパーソナリティに従って行動してください。
        """;
        _aiChatClient = aiChatClient;
    }
    public async Task<SageResponse> VoteAsync(VoteOption option)
    {
        var messages = new List<ChatMessage>
        {
            new ChatMessage(ChatRole.System, _systemPrompt),
            new ChatMessage(ChatRole.User, GetUserMessage(option))
        };

        var result = await _aiChatClient.GetResponseAsync<SageResponse>(messages, _voteChatOption);
        /// ....
        return sageVoteResponse;
    }


    private string GetUserMessage(VoteOption voteOption)
    {
        return $"""
        ## Topic
        次の議題について投票してください。
        --------
        {voteOption.Topic}
        --------
        Yesの判断基準: {voteOption.YesCriteria}
        Noの判断基準: {voteOption.NoCriteria}
        """;
    }
}
```

これらの賢者を`MagiService`クラスで管理し、議題を投げ込むと合議して結果を返すようにしています。
```csharp
public class MagiService
{
    // ... 省略 ...
    public async Task<MagiResponse> MajorityVoteAsync(VoteOption option)
    {
        List<Task<SageResponse>> voteTasks = new();

        foreach (var sage in _sages)
        {
            voteTasks.Add(sage.VoteAsync(option));
        }

        var results = await Task.WhenAll(voteTasks);

        var response = new MagiResponse(
            FinalDecision: DetermineFinalDecision(results),
            CountOfYes: results.Count(r => r.VoteResult == VoteEnum.Yes),
            CountOfNo: results.Count(r => r.VoteResult == VoteEnum.No),
            YesReasons: results.Where(r => r.VoteResult == VoteEnum.Yes).Select(r => r.Reason).ToList(),
            NoReasons: results.Where(r => r.VoteResult == VoteEnum.No).Select(r => r.Reason).ToList()
        );

        return response;
    }

    private FinalDecisionEnum DetermineFinalDecision(SageResponse[] results)
    {
        int yesCount = results.Count(r => r.VoteResult == VoteEnum.Yes);
        int noCount = results.Count(r => r.VoteResult == VoteEnum.No);

        if (yesCount > noCount)
            return FinalDecisionEnum.Yes;
        else if (noCount > yesCount)
            return FinalDecisionEnum.No;
        else
            return FinalDecisionEnum.Tie;
    }
}
```

デフォルトでは元ネタに登場するパーソナリティを持つ賢者を3人用意しています。
しかし、タスクによっては異なるパーソナリティを持つ賢者が必要な場合もあると思いますので、コンストラクタで`ReadOnlyCollection<Sage>`を受け取るオーバーロードも用意しています。
```csharp
public class MagiService
{
    private readonly ReadOnlyCollection<Sage> _sages;

    public MagiService(IChatClient aiChatClient)
    {
        _sages = new ReadOnlyCollection<Sage>(
        [
            new Sage("論理・客観的な分析に偏る判断", aiChatClient),
            new Sage("慎重で保守的、リスク回避型", aiChatClient),
            new Sage("感情的・直感的な判断や妥協を提示", aiChatClient)
        ]);
    }

    public MagiService(ReadOnlyCollection<Sage> sages)
    {
        _sages = sages;
    }

    // ... 省略 ...
}
```

## 動かしてみる
リポジトリにコンソールアプリとWebアプリのサンプルコードを用意しているのでそれを使って実際に動かしてみます。
![](/images/llm-magi-system/sample_ui.png)

### 夏目漱石の翻訳は正しいか？
![](/images/llm-magi-system/study1.png)
夏目漱石が英語の「I love you」を「月が綺麗ですね」と翻訳したことは有名です。この翻訳について、賢者たちに意見を聞いてみましょう。
![](/images/llm-magi-system/study1_result.png)
ご覧の通り、賢者たちの意見は分かれましたが、多数決としては否決されました。


### 猫と犬ではどちらが人気か？
![](/images/llm-magi-system/study2.png)
次に日本国内での猫と犬の人気について賢者たちに意見を聞いてみます。これはWeb検索やDeep Researchなどを使って調べたほうが良いかもしれませんが、あえてLLMに聞いてみます。
![](/images/llm-magi-system/study2_result.png)
なんということでしょう（私は犬派です）。猫の方が人気があることが可決されてしまいました・・・。

### 「芋煮」という料理は牛肉醤油味の料理だよね？
![](/images/llm-magi-system/study3.png)
最後に、 **自明ではありますが、** 芋煮という料理は牛肉醤油味の料理であるかどうかを賢者たちに聞いてみます。
![](/images/llm-magi-system/study3_result.png)
~~なんでやねん！あれは豚汁やないかい！~~

## リポジトリ
ご興味がある方はぜひリポジトリを覗いてみてください。
https://github.com/07JP27/MagiSystem
