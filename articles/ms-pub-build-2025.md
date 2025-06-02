---
title: "Microsoft (有志)メンバーが選ぶ！Microsoft Build 2025注目セッション"
emoji: "💁‍♂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "microsoft"]
published: true
publication_name: "microsoft"
---

Microsoftの年次フラグシップイベントであるBuildが今年も2025/5/20～23に開催され、数多くの新情報の発表や製品活用のヒントとなるセッションが行われました。
この記事ではその数多くのセッションの中から、[Microsoft (有志)](https://zenn.dev/p/microsoft)Publicationのメンバーがそれぞれの専門分野の観点から特に注目すべきセッションを選定しました。Microsoft (有志)Publication初のコラボ記事です！


## 発表の全体像
Microsoftのフラグシップイベントでは、毎回Book of Newsと呼ばれる新情報サマリーが公開され、発表の全体像を把握することができます。
https://news.microsoft.com/build-2025-book-of-news/ja/

ただ、サマリーと言っても情報が大量にあります。MicrosoftはIaaSからSaaSまで幅広い製品を提供しているため、全ての情報を把握するのは大変です。
そんな方向けに、情報をさくっとキャッチアップできるようにNotebookLMでBook of Newsをソースに音声概要を生成しました。こちらもぜひご活用ください。
https://notebooklm.google.com/notebook/081a2f92-9534-4c30-9421-76052ac9a751/audio

それでは各メンバーが選んだ注目セッションを紹介していきます!
👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇

## [Yoshihiro  Matsumoto（yomatsum）](https://zenn.dev/yomatsum)
私からは Build 2025 内で王道っぽい雰囲気のするものを取り上げてみたいと思います。
### Azure AI Foundry: The AI app and Agent Factory | BRK155
AI の最新技術を単に紹介するだけでなく、「どのようにすれば AI を使って実際に価値を生み出し、日々の作業をより良くできるのか」という問いに対する、具体的で実践的な解を示している点がよいと思います。機能や性能で勝負するフェーズから、業務活用のフェーズに移ってきているなと感じられるセッションかなと思います。
https://www.youtube.com/watch?v=DUdRdeUtuZQ

### Best practices for building agentic apps with Azure AI Foundry | BRK152
初日の Keynote で紹介されていたエージェント デモ（イベント会場を探して、カレンダーで招待、SNSで共有）を実際に作った過程を共有している点がよいと思います。ものすごく美しいベスト プラクティスを話すというよりも、実際の開発で何が起こったのかを見せた上で対処が共有される、ライブ感があるセッションだと思います。金科玉条をお求めの方よりは、黎明期のエージェント開発の苦労話を他山の石としたい方にお勧めします。
https://www.youtube.com/watch?v=WH9iBu9hT4E

## [Miho Kurimoto（mihohoi）](https://zenn.dev/mihohoi)
### Streamlining AKS Debugging: Techniques to solve common & complex problems | BRK181
AKS に問題があった場合、どこからどうやって見ていけばいいの？というのを障害パターンをいくつかセッションの中でコマンド叩きながら解消して見せてくれたのが熱い。AKS に関わっている方はぜひ。
https://www.youtube.com/watch?v=HMCM49mIr5I&list=LL

### A 10x Faster TypeScript with Anders Hejlsberg | BRK116
Build の動画の中で、異様に再生回数が多い動画が2つあります。一つは「[Learn how to write better C# code in 15 minutes | DEM515](https://youtu.be/-Jyj4xlwss8?si=vr69bq3GirRARvSU)」、もう一つがこの動画です。TypeScript のコンパイラとツールを Go でネイティブに移植するプロジェクトの取り組みについて、選定理由からどこが良かった...という内容をマニアックに語っています。内容も素晴らしいですが、取り組み内容を伝えている熱い姿勢に見とれてしまいます。
https://youtu.be/UJfF3-13aFo?si=MFnHChwn9Rfd_9GY

## [Kazuki Ota（okazuki）](https://zenn.dev/okazuki)
### Python Meets .NET: Building AI Solutions with Combined Strengths | BRK115
Python 使いと C# 使いの掛け合いが面白い…！！
https://www.youtube.com/watch?v=fDbCqalegNU

### Build the next gen of AI apps with .NET: Models Data Agents & More | BRK104
.NET で AI 開発をするときに何を使って作ればいいのかが全部わかる！
https://www.youtube.com/watch?v=gBIhVOg2aPc

## [Kazuki Komori（kazuyan）](https://zenn.dev/kazuyan)
### Under the hood and into the magic of GitHub Copilot | BRK108
GitHub Copilot Coding Agent の機能の紹介です！Issue を作るだけで人間と同じような開発フローで TODO タスクを洗い出してコードを書いてくれます。
https://www.youtube.com/watch?v=EzP2HTD6wRM

### Building agents in Copilot Studio using Model Context Protocol open standards and more | BRK158
Copilot Studio で MCP を使う紹介です！ローコードツールで外部サービスとの連携がしやすくなったのもポイント高いですね！
https://www.youtube.com/watch?v=7AHj1azyPwc

## [Nobufumi Murata（nmurata113）](https://zenn.dev/nmurata113)
### Shift Left: Secure Your Code and AI from the Start | BRK230
AI 開発をする上でより重要になってきているセキュリティ。Design から Run/Monitor までの開発サイクルの中でより早い段階(Shift Left) でセキュリティを取り込んでいく事が必要になってきていますが、その考え方などが体系的に学べます。
https://www.youtube.com/watch?v=-cu9S7mGmBY

## [Masato Ueda（uedait）](https://zenn.dev/uedait)
### Inside Azure innovations with Mark Russinovich | BRK195
Azure の CTO である Mark Russinovich が、 Azure のインフラストラクチャや、Azure Boost、Confidential Computing など、Azure を支えるための様々な技術の進化について解説しているセッションです。 Build は開発者向けのセッションではありますが、その中でも珍しく Cloud Platform 全体について触れているセッションです。
https://www.youtube.com/watch?v=lHBo_lDWFcI

## セッションアーカイブ一覧
全てのセッションは以下のリンクからアーカイブ動画を視聴できます。
https://build.microsoft.com/en-US/sessions