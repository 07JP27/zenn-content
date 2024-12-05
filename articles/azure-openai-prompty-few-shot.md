---
title: "PromptyでもFew-shotプロンプティングができます"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Azure', 'OpenAI', 'Prompty']
published: true
publication_name: "microsoft"
---

PromptyはMicrosoftがOSSで開発をしているLLMのプロンプトを管理するファイルフォーマットです。
VS Code拡張機能など周辺ツールのエコシステムによってプロンプトのテスト〜デプロイまでを支援します。詳しくは以下でとりあげています。

https://zenn.dev/microsoft/articles/azure-openai-prompty


## なぜできないと思っていた？
結果的にはタイトルの通りできたのですが、なぜできないと思っていたかというと、VS CodeでPromptyを開発する場合、systemやuserはコードハイライトがされますが、assistantだけはされません。
ここからsystemとuserのプロンプトを書くことはできるが、assistantのプロンプトを書く、つまりFew-Shotのようなことはできないと思っていました。

![](/images/azure-openai-prompty-few-shot/vscode.png)

## なぜできるとわかった？
PromptyのGitHubのissueで聞いたところ一晩でコメントをもらえました。

https://github.com/microsoft/prompty/issues/140


## 実際にやってみると
以下の通り、`assistant:`で指定することでFew-shotプロンプトを書くことができました。

assistant:でプロンプトを書いて
```prompty
system:
You are an AI assistant supporting a user named "{{userName}}". 

user:
Hello

assistant:
Hi

user:
{{question}}
```

pythonから読み取って
```python
messages = prompty.load("./basic.prompty")
print(prompty.prepare(messages, {"userName": "John", "question":"What's up?"}))
```

messagesを表示。assistantロールも認識されている。
```text
[
  {'role': 'system', 'content': 'You are an AI assistant supporting a user named "John".'},
  {'role': 'user', 'content': 'Hello'}, 
  {'role': 'assistant', 'content': 'Hi'},
  {'role': 'user', 'content': "What's up?"}
]
```

## まとめ
あまり内容が無いような記事になってしまいましたが、ツール側の不具合と思われる現象も相まって同じ道を踏む方がいらっしゃるかもしれないので、VS Codeの拡張機能が更新されることを祈りつつ、インターネットの海に書き残しておきます。