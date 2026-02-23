---
title: "GitHub Copilotを使って英語学習をしよう（英語以外もOK）"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["english", "githubcopilot"]
published: true
publication_name: "microsoft"
---

昨今コーディングエージェントが汎用的なタスク実行ツールとして使われるケースが増えてきました。GitHub Copilotも例外ではなく、コード生成だけでなく、様々なタスクを実行するためのツールとして活用できるようになっています。
そこで今回は~~私がスランプに陥っている~~英語学習を題材にGitHub Copilotを使ってコーディングだけではない学習タスクを補助する方法について紹介します。

![image](/images/github-copilot-for-learning/result.png)

## 構成
以下のリポジトリを見ていただくのが一番わかりやすいと思います。
https://github.com/07JP27/EnglishLearningWorkspace


ファイルツリーは以下のようになっており、GitHub Copilotの設定と学習履歴が大部分を占めています。
```
EnglishLearningWorkspace/
├── .github/
│   ├── copilot-instructions.md　：GitHub Copilotに対するベースの指示が記載されたファイル
│   └── prompts/                 ：各テストのプロンプトが格納されたディレクトリ
│       ├── collocationTest.prompt.md
│       ├── generalTest.prompt.md
│       ├── grammarTest.prompt.md
│       ├── longTextTest.prompt.md
│       └── vocabularyTest.prompt.md
├── .gitignore
├── README.md
└── learninghistory/　：学習履歴が保存されるディレクトリ
    ├── Collocation.md
    ├── Grammar.md
    ├── LongText.md
    └── Vocabulary.md
```

## ポイント
### その１：1コマンドでテスト実施
GitHub Copilotでは「プロンプト」を定義することにより、いわゆるスラッシュコマンドとしてそのプロンプトが実行できます。
VS Codeのチャットウインドウで「/」と入力すると、定義されたプロンプトが補完されるので定義したコマンドを選んでエンターを押すだけで、テストがはじまります。
![image](/images/github-copilot-for-learning/commnad.png)

### その2：AskUserQuestionsツールでの回答
今までもLLMを使って4択問題に回答させることはできましたが、その場合選択肢の数字やアルファベットを入力して送信する必要がありました。
現在ではAskUserQuestionsツールを使うことで、ユーザーに選択肢や入力欄を提示してクリックなどの最低限のインタラクションで回答できるようになりました。これにより、よりUXに優れた形でテストを受けることができます。
![image](/images/github-copilot-for-learning/ask.png)

### その3：学習記録のGit管理
LLMモデルを使った学習の場合、スポット的になってしまうことが多いです。
つまりLLMモデル単体ではあなたの学習履歴（コンテキスト）をメモリとして保持していないため、テストを受けるたびに同じ問題が出てきたり、問題がやたら簡単だったり難しかったりすることがあります。
この問題を解決するためにワークスペース内に学習履歴を保存するディレクトリを作成し、テストの結果をMarkdown形式で保存するようにしました。2回目以降テストを行う場合はこの履歴を読み込んでから問題を考えるため、反復学習や問題のレベルにばらつきがなくなります。
さらに、GitHubのGitバージョン管理機能を活用して学習履歴を管理できるため、過去の学習履歴を簡単に振り返ることができ、学習の進捗を把握しやすくなります。
![image](/images/github-copilot-for-learning/history.png)

### 応用編：GitHubの機能を活用して学習計画を立てたり、他の人と共有
GitHubにはIssuesやProjectsなどの機能があるので、それらを活用して学習計画を立てたり、他の人と共有することもできます。例えば、Issuesを使って学習したいトピックや問題を管理し、Projectsを使って学習の進捗を可視化することができます。また、リポジトリを公開することで、他の人と学習成果を共有したり、フィードバックをもらうこともできます。
すべてをGitHub上で管理することで、過去の振り返りから未来の計画まで一元的に行えるようになります。


## まとめ
このようにGitHub CopilotとGitHubの機能を組み合わせることで、英語だけでなく、さまざまな学習がより効果的に行えるようになります。
たとえばプログラミングや資格の学習に応用することもできますし、歴史や科学などの知識を深めるためのテストを作成することもできます。アイデア次第で、無限の可能性が広がります。

興味がある方はぜひ以下のリポジトリをクローンして、学習履歴ファイルをリセットしてから自分の学習に活用してみてください。
https://github.com/07JP27/EnglishLearningWorkspace/settings