---
title: "ChatGPTをもっと便利に！1つのChatGPTボットでプロンプトをスイッチしよう"
emoji: "🔀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'openai', 'teams', 'chatbot', 'chatgpt']
published: true
publication_name: "microsoft"
---

ChatGPTを使わない日はない、という方も少なくないのでは無いでしょうか。私もその一人です。
ある日ChatGPTを使っていて、ふと思いました。「なんでChatGPTとチャットするために別でWeb UIを起動する必要があるんだ・・・」
そう、普段人間とチャットしているツールに集約したいんです。SlackやTeamsなどのチャットツールをUIに使えばいいんです。
そう思い、単純にChatGPTをTeamsボットとして組み込んで使っていました。

**が**

「毎回毎回インストラクションを書くのがめんどくさい・・・GPTs使いたい・・・」と思うようになりました。
というわけで本記事では以下の条件を満たすChatGPTボットを作ってみたので方法ついてご紹介します。

- UIにSlackやTeamsなど使い慣れたチャットツールを使う
- GPTsのような仕組みを作り、複数の設定を1つのボットの中で切り替えて使えるようにする(とりあえずRetrievalやCode Interpreterは無しでシステムプロンプトのみ)

## できたもの↓
![](/images/system-prompt-switching-gpt-bot/character.gif)
コマンドを打つとそれに対応したシステムプロンプトが自動的に適用されて、ユーザーは同じ文章を入力してもシステムプロンプトに応じて返答が変化していることが見て取れます。


サンプルソースコードはこちらに置いています。
https://github.com/07JP27/SystemPromptSwitchingGPTBot


# どう実現しているか？
以降はTeamsの前提で話を進めますが、考え方は他のチャットツールでも同じです。
Teamsボットには「コマンド」という機能があります。通常のチャットエリアにカーソルを合わせるとそのボットで使えるコマンドが表示されます。
これを使ってシステムプロンプトを切り替えようというアプローチです。
![](/images/system-prompt-switching-gpt-bot/command.png)
この **「コマンド」はあくまでもTeams側の入力サジェストに表示されるかどうかだけです。ボットから見れば特別な何かというわけではなく、ただのテキストとして渡されてきます。**
つまり、ボット側で入力されてきたテキストがコマンドかどうかを判定してシステムプロンプトを切り替えるという実装になります。そのため別のチャットツールの場合でも平文テキストとして扱えばOKですし、Slackであればスラッシュコマンドに対応させる、という方法もあると思います。
細かい一部の処理は省略していますが、ボットの主なフロー+αは以下のようになります。
![](/images/system-prompt-switching-gpt-bot/flow.png)


上記のフローでは２つのストアがありますが、今回はなるべく簡単にするために、システムプロンプトはハードコードで保持しています。
また、会話の状態はメモリストアに保持しています。永続化したい場合でもBot Frameworkを使用しているのでリソースさえデプロイしてしまえばCosmos DBやAzuew Storageに[簡単に切り替えられる](https://learn.microsoft.com/ja-jp/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0#storage-layer)ようになっています。

![](/images/system-prompt-switching-gpt-bot/classmap.png)

リソースについても今回は外部データの利用（Retrieval）は実装していないので、Azure OpenAIとボットに関連するサービスのみのデプロイになります。
![](/images/system-prompt-switching-gpt-bot/arch.png)


## GPTsに相当するカタログを作る
個々のGPTsに相当する部分を定義していきます。実際にはシステムプロンプトだけではなく、システムプロンプトの用途に応じて他のパラメーターも定義できるようにしました。
デバッグなどロジカルに出力してほしいときもあれば思考の壁打ちのように多様な出力をしてほしいときもあるので、それらに対応できるようにしています。

```cs:app/GptConfiguration/Default.cs
public class Default:IGptConfiguration
{
    public string Id => "default";
    public string Command => "default";
    public string DisplayName => "デフォルト";
    public string Description => "デフォルトのシステムプロンプト";
    public float Temperature => 0.0f;
    public int MaxTokens => 400;
    public string SystemPrompt => "You are an AI assistant. Help users accomplish their goals as quickly and easily as possible.";
}
```

そしてStartup.csでDIコンテナに登録します。金太郎飴的に`IGptConfiguration`のインターフェースを実装すればいくらでも増やせるような実装になっています。

```cs:app/Startup.cs
var catalog = new List<IGptConfiguration>
{
    new Default(),
    new Korosuke(),
    new Doraemon(),
    new Translate(),
    new CodeReview(),
};
services.AddSingleton(catalog);
```

## コマンドを認識してシステムプロンプトを切り替える
```cs:app/Bots/GPTBot.cs
string inputText = turnContext.Activity.Text;
            
if(inputText.StartsWith("/"))
{
    string command = inputText.Trim().ToLowerInvariant().Substring(1);
    //[省略]
    var systemPrompt = _systemPrompts.FirstOrDefault(x => x.Command == command);

    if(systemPrompt != null)
    {
        // systemPromptのSystemPromptを返す
        var switchedMessage = $"会話履歴をクリアして、**{systemPrompt.DisplayName}**モードに設定しました。\n\nこのモードでできること：{systemPrompt.Description}";
        await turnContext.SendActivityAsync(MessageFactory.Text(switchedMessage, switchedMessage), cancellationToken);
        conversationData.CurrentConfigId = systemPrompt.Id;
        conversationData.Messages = new (){new GptMessage(){Role = "system", Content = systemPrompt.SystemPrompt}};
        return;
    }
    //[省略]
}
```


## Teamsのマニフェストを作る
Teamsボットのマニフェストファイルで、コマンドとしてサジェストされる文字列を`commandLists`の`commands`に定義します。
```json:manifest/manifest.json
"commandLists": [
    {
        "commands": [
            {
                "title": "/korosuke",
                "description": "コロ助のように喋ります"
            },
            {
                "title": "/doraemon",
                "description": "ドラえもんのように喋ります"
            },
            {
                "title": "/default",
                "description": "デフォルトのシステムプロンプト"
            },
            {
                "title": "/clear",
                "description": "会話履歴をクリアします"
            },
            {
                "title": "/codereview",
                "description": "コードをレビューします"
            },
            {
                "title": "/translate",
                "description": "入力された文章を翻訳します"
            }
        ],
        "scopes": [
            "personal"
        ]
    }
],
```


# ユースケース
冒頭のデモよりも、もう少し真面目なユースケースとして筆者がChatGPTによく依頼する２つのタスクを紹介します。

## コードレビュー
コードレビューのシステムプロンプトの設定は[こちら](https://github.com/07JP27/SystemPromptSwitchingGPTBot/blob/main/app/GptConfiguration/CodeReview.cs)になります。コードレビューモードに切り替えてコードだけを投げればちゃんとレビューをしてくれます。
![](/images/system-prompt-switching-gpt-bot/review.gif)



## 翻訳
翻訳も筆者がよく使うタスクの一つです。特に翻訳の場合、メール文面なのかチャットなのか、カジュアルなのかフォーマルなのかといったケースが多いので、用途に切り替えたシステムプロンプトを用意することによって手間が少なくなる代表的なタスクかなと思います。翻訳のシステムプロンプトの設定は[こちら](https://github.com/07JP27/SystemPromptSwitchingGPTBot/blob/main/app/GptConfiguration/Translate.cs)。
![](/images/system-prompt-switching-gpt-bot/translate.gif)



# 発展性
今回紹介したものが「今後どのように拡張できるのか」「どのような場面で強さを発揮するのか」という点については以下のようなことが考えられます。

## RetrievalRetやCode Interpreterの実装
今回はシステムプロンプトのみの実装ですが、RetrievalやCode Interpreterの実装も可能です。
各システムプロンプトの設定クラス内に検索処理や実行処理、Completionなどをそれぞれ定義してそれを呼び出すようにすれば実現可能です。
![](/images/system-prompt-switching-gpt-bot/like-gpts.png)


## RAGの範囲指定
外部情報を検索してその結果を加味して回答を生成する「RAG」ですが、GPTの回答精度だけではなく、検索エンジンの検索精度も重要な指標になります。
たとえば、社内情報をまるっと検索できることは非常に汎用的である反面、検索タスクに適用するシナリオが膨大になってしまいます。そうなると検索対象におけるノイズが増えてしまい、検索精度が下がる懸念があります。
そこで今回の仕組みを使って /hrや/finunceなどのコマンドを作り、検索対象を絞り込む(モードによって検索対象のインデックスを切り替える)ことで検索精度を上げることが期待できます。
![](/images/system-prompt-switching-gpt-bot/separate.png)


# まとめ
本記事ではChatGPTボットと使い慣れたチャットUI上でシステムプロンプトを切り替えながら会話する方法についてご紹介しました。
ボットとのタッチポイントが１つで済むようになり、シンプルでありながらで様々な振る舞いができるようになるので、ChatGPTとの協業がますますやりやすくなりそうです。

サンプルソースコードはこちら
https://github.com/07JP27/SystemPromptSwitchingGPTBot