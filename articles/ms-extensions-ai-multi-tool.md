---
title: "Microsoft.Extensions.AIで複数のAIFunctionをChatOptionsのToolsに登録する"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["dotnet", "csharp", "ai", "microsoft-extensions-ai"]
published: true
publication_name: "microsoft"
---

## 何が問題か
かなり細かい話題ですが、Microsoft.Extensions.AIを使用してC#でTool Callingのツール（関数）を定義使用とする場合、`AIFunction`が使えます。
```csharp
List<AIFunction> tools =  new List<AIFunction>{
    AIFunctionFactory.Create(Hoge),
    AIFunctionFactory.Create(Fuga),
    AIFunctionFactory.Create(Piyo)
};
```

そしてそれを`ChatOptions`に登録する際、以下のように書くのが直感的ではありますが、残念ながらこの書き方はエラーになります。

```csharp
List<AIFunction> tools =  new List<AIFunction>{
    AIFunctionFactory.Create(Hoge),
    AIFunctionFactory.Create(Fuga),
    AIFunctionFactory.Create(Piyo)
};

ChatOptions options = new ChatOptions
{
    Tools = tools,
};
```

:::message alert
Cannot implicitly convert type 'System.Collections.Generic.List<Microsoft.Extensions.AI.AIFunction>' to 'System.Collections.Generic.IList<Microsoft.Extensions.AI.AITool>'. An explicit conversion exists (are you missing a cast?)CS0266
:::

なぜならChatOptionsのToolsは`IList<AITool>`型であるのに対して、toolsは`List<AIFunction>`だからです。
AIFunctionはAIToolを継承していますが、C#の型システム上、`List<AIFunction>`は`IList<AITool>`に暗黙的に変換できません（IList<T>は共変性を持たないため、List<AIFunction>をIList<AITool>に直接代入できない）。

https://learn.microsoft.com/ja-jp/dotnet/csharp/programming-guide/concepts/covariance-contravariance/

## 解決策

### 選択1：配列でツールを定義する
上記のドキュメントにもある通りC#では配列型は共変であるため、`AIFunction[]`型の配列は`AITool[]`型に暗黙的に変換できます。
```csharp
 private static AIFunction[] tools =  new AIFunction[]{
    AIFunctionFactory.Create(Hoge),
    AIFunctionFactory.Create(Fuga),
    AIFunctionFactory.Create(Piyo)
};

private ChatOptions options = new ChatOptions
{
    Tools = tools,
};
```

### 選択2：配列を展開し、ターゲット型推論を使用する
配列を使用していると、実行中に動的にツールを出し入れする処理が書きづらい場合があります。
その場合、ターゲット型推論を利用して、`ChatOptions`のToolsプロパティに直接配列を展開する方法があります。
```csharp
List<AIFunction> tools =  new List<AIFunction>{
    AIFunctionFactory.Create(Hoge),
    AIFunctionFactory.Create(Fuga),
    AIFunctionFactory.Create(Piyo)
};

private ChatOptions options = new ChatOptions
{
    Tools = [..tools],
};
```

この記法は内部的に以下の式のようになります。
```csharp
Tools = new AITool[] { tools[0], tools[1], tools[2] }
```

ただしこの記法は参照の共有ではなくコピーが発生します。この処理を大量のに行う場合はパフォーマンス的に非効率になる可能性がありますので注意してください。