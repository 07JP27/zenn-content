---
title: "Microsoft (有志)メンバーが選ぶ！Microsoft Ignite 2025注目セッション"
emoji: "💁‍♂️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["azure", "microsoft"]
published: true
publication_name: "microsoft"
---

Microsoftの年次フラグシップイベントであるIgniteが今年も2025/11/18～21の期間で開催され、数多くの新情報の発表や製品活用のヒントとなるセッションが行われました。
特にAzure上でもClaudeのモデルが使えるようになるアナウンスはイベント前から大きな話題となっていました。

この記事ではその数多くのセッションの中から、[Microsoft (有志)](https://zenn.dev/p/microsoft)Publicationのメンバーがそれぞれの専門分野の観点から特に注目すべきセッションを選定しました。


## 発表の全体像
Microsoftのフラグシップイベントでは、毎回Book of Newsと呼ばれる新情報サマリーが公開され、発表の全体像を把握することができます。

https://news.microsoft.com/ignite-2025-book-of-news



それでは各メンバーが選んだ注目セッションを紹介していきます!
👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇

## [Ryosuke Otaka](https://zenn.dev/ryo117)
Microsoft Ignite 2025 全体を眺めていて、多くの企業が AI エージェント活用を検証段階から本格運用へ移しつつあるように私には感じられました。こうした流れの中で、次のステップとして示されていたのが **“Unified knowledge layer for agents (エージェントのための統合ナレッジレイヤー)”** で、これはエージェントに対し複数のナレッジソースを意識させず、単一のナレッジエンドポイントとして統合的に提供する共通基盤を指す概念とされています。情報取得を Retriever に任せる Foundry IQ と Fabric IQ はまさにこの方向性を体現しており、従来個別に構築していた RAG を統合レイヤーへ集約し、エージェントがビジネス文脈を踏まえて自律的に知識を取得できる、より高度なナレッジソースの抽象化へ向けたパラダイムシフトが進んでいると感じました。

### Foundry IQ: the future of RAG with knowledge retrieval and AI Search（BRK196）
Microsoft Foundry（旧称 Azure AI Foundry）ポータルでの Foundry IQ 作成デモに沿って、断片化されたデータソースをシームレスに統合する仕組み、開発者体験、基盤技術である Agentic Retrieval、そして Foundry IQ の検索性能について解説されています。
https://ignite.microsoft.com/en-US/sessions/BRK196

### Build agents with knowledge, agentic RAG and Azure AI Search（BRK193）
Azure AI Search の全体像のおさらい、現実世界での RAG の課題とハイブリッド検索のみでのアプローチの限界、解決策としての Agentic Retrieval の機能解説とデモが含まれています。
https://ignite.microsoft.com/en-US/sessions/BRK193

### Microsoft Fabric IQ: Turning unified data into unified intelligence (BRK222)
Fabric IQ の機能紹介デモビデオを交えながら、現実世界で数値データの探索・活用を行う際の課題、解決策としての Fabric IQ およびその中核となるオントロジー、Fabric IQ の活用事例、Foundry IQ との連携などについて解説されています。
https://ignite.microsoft.com/en-US/sessions/BRK222


## [Shunsuke Yoshikawa](https://zenn.dev/ussvgr)
これまで行われてきた各サービスのアップデートが、AI の力でどんどん統合されていったという印象です。

### What’s new and what’s next in Azure IaaS（BRK 171）
クラウドの基本は IaaS 、ということで IaaS 関連のアップデートのおさらいです。AKS もあるよ。
https://ignite.microsoft.com/en-US/sessions/BRK171

### Advanced capabilities and innovation of Azure Storage solutions（BRK 159）
すべての基礎にストレージあり、ということでストレージに関する進化を学びましょう。
https://ignite.microsoft.com/en-US/sessions/BRK159

### From zero to Kubernetes with AKS Automatic（BRK 121）
Azure Kubernetes Service って難しそう、と敬遠しているあなたにお勧めのセッション。AKS Automatic で楽々運用しちゃいましょう。個人的見どころは、[GitHub リポジトリを指定するだけの楽々デプロイ](https://learn.microsoft.com/ja-jp/azure/aks/automatic/quick-automatic-from-code?source=recommendations&tabs=generate-dockerfile%2Cgenerate-kubernetes-manifests%2Cnew-aks-automatic-cluster)、[マネージドシステムノードプール](https://learn.microsoft.com/ja-jp/azure/aks/automatic/aks-automatic-managed-system-node-pools-about)、[マネージド 名前空間 ](https://learn.microsoft.com/ja-jp/azure/aks/concepts-managed-namespaces)あたりです。
https://ignite.microsoft.com/en-US/sessions/BRK121

### Scott & Mark learn to connect the dots（BRK 432）
ベテランエンジニア 2 人がコンピューターの進化を面白おかしく語りあう会です。自分も将来こんな風に楽し気に昔話ができるようになりたいなと思いました。毎回恒例の Mark Russinovich によるモンスターマシンのタスクマネージャー芸、今回は [Altair 8080](https://en.wikipedia.org/wiki/Altair_8800) でした。
https://ignite.microsoft.com/en-US/sessions/BRK432

## [Yurie Mori](https://zenn.dev/yuriemori)
この前のBuildで出たAgentic DevOpsがDefender for CloudとGitHubの統合、SRE Agentのサブエージェント、Coding Agent × MCPとかSRE Agent×MCPとかでだいぶ具体化されてきた感じ。そしてGitHub Copilot ×AzureやGitHub × Defender for Cloudなど、GitHubとその他のMicrosoft Techonologyの統合が進んできてる。これまではDevOpsのDev側に体験の改善のための機能が集中してた感があったけど、今回のアップデートでDevOpsのOps側のmodernizeも進んできて、DevOpsの包括的な変革が進んでる感じがとてもエキサイティング。

GitHub, Azure DevOps関連のセッションの詳細なまとめはこちらから:
https://zenn.dev/yuriemori/articles/a748d9bd900ca5

### AI-powered workflows with GitHub and Azure DevOps（BRK106）
- Azure DevOpsとGitHubどっちを使うべきなの？Azure DevOpsはもう使わない方がいいの？という昨今の議論に対する直接的なアンサーセッション。
- Azure DevOpsとGitHubの具体的な併用のプラクティスが紹介されている。Azure BoardsからCoding Agentを使うデモあり。

https://ignite.microsoft.com/en-US/sessions/BRK106

### Inside Microsoft's AI transformation across the software lifecycle（BRK115）
- 「Microsoft自身」はAIをどのようにSDLCに活用してきたのか&しているのか？というお話。
- Microsoft社内ではGitHub Copilot, Coding Agent, SRE Agent, GitHub Copilot Code Reviewなどのツールをどのように活用し、Agentic DevOpsをどのように実現しているか？そしてその各ツールをSDLCに取り入れたことによって、具体的にどのような効果があったか？そして改革の中でAI Agentの振る舞いが期待したものでなくてがっかりして行き詰った時にどのように乗り越えたのか？
- AI Agentとの付き合い方やMicrosoftにおけるPlatform Engineeringの取り組み方に興味がある方、コンセプトで終わらない具体的なAgentic DevOpsの実践方法に興味がある方はぜひ。

https://ignite.microsoft.com/en-US/sessions/BRK115

### Secure code to cloud with AI infused DevSecOps(BRK112)
- 今回発表されたDefender for Cloud × GitHub Advanced Securityの統合（Public Preview）で具体的になにができるの？ということを示している。
    - [Security Where It Matters: Runtime Context and AI Fixes Now Integrated in Your Dev Workflow | Microsoft Community Hub](https://techcommunity.microsoft.com/blog/appsonazureblog/security-where-it-matters-runtime-context-and-ai-fixes-now-integrated-in-your-de/4470794)
- 脆弱性はたくさん検知できるようになった、でも修正する暇がない、そしてAIの登場によって生産されるコード（検査すべき）対象はどんどん増える、というDevSecOpsの課題に真っ向から向き合う内容。
- セキュリティのシフトレフトをGHASが、シフトライトをDefender for Cloudが担うイメージ。コンテナイメージの脆弱性やランタイムアプリケーションの脆弱性検知と修正はGHASだけでは手が届かなかった領域なので、これによってDevSecOpsの完成度がかなり高くなりそう。

https://ignite.microsoft.com/en-US/sessions/BRK112


### Secure, compliant, and fast with GitHub(BRK108)
- 企業でGitHub Enteprirseを安全に、そして開発体験を損なわずにガバナンスするTipsが紹介されている。Enterprise teams, Organization Custom Property、Rulesetなどのエンタープライズ層の機能を使って、開発基盤としてのGitHubをどのように整備、運用していくか、開発者の開発体験を損なわずに安全性を保つためには管理者はどのようにすればよいのかをデモ付きで紹介してくれる。
- GitHub Enterpriseのガバナンスの設計思想を体系化したPlatform Foundation、GitHub Enterpriseの管理/運用の最新の具体的なTipsが豊富に紹介されている。GitHub管理者の方/Platform Engineer/DevOps Engineerの方は必見！

https://ignite.microsoft.com/en-US/sessions/BRK108

### Ship faster. Stress Less. Idea to ops with Azure and GitHub Copilot(BRK109)
- GitHub Copilotや様々なエージェント、MCPサーバーを駆使してAzure上で動かすアプリケーションのデザインから運用までを見せてくれるデモが圧巻。GitHub Copilot × Azureに興味がある方は必見。
- Spek Kit × Azure MCP Serverによる設計や、Azure SRE Agentのサブエージェントを使って各種タスクに特化したエージェントやサードパーティ製品のMCPサーバーを協業させてインシデント対応を行うデモはかなり見ごたえあり。

https://ignite.microsoft.com/en-US/sessions/BRK109

### Building Sustainable Software with Agentic DevOps on GitHub(BRK218)
- Agentic DevOpsはどのようにSustainablityに貢献できるか？という話。電力消費を最小化するためのプロセス設計や技術選択によって効率が向上するだけでなく炭素排出量の削減にもコミットできますよという内容。
- DevOps×Sustainabilityというトピックは新鮮でとてもよかった。

https://ignite.microsoft.com/en-US/sessions/BRK218

## [Naoki Matsumoto](https://zenn.dev/chips0711)
生成AIブームが始まってからまだ（もう？）約3年のなか、今回のIgniteでは、CopilotとAIエージェント（Agent 365、Work IQなど）をあらゆる製品レイヤーに組み込み、業務プロセスそのものをエージェント前提に作り替えていく「Frontier Firm（人間主導＋エージェント運用の企業）」像をMicrosoftの回答として新たに提示した節目となるイベントでした。そんな感想を抱きつつ、以下の3つのセッションを紹介します！

### AI fine-tuning in Microsoft Foundry to make your agents unstoppable（BRK 188）
Microsoft Foundryを活用し、AIエージェントを「安く・速く・賢く」するファインチューニング技術の解説セッションです。特に、学習データをAIが自動生成する「Synthetic Data Generation（合成データ生成）」という一見地味な新機能が登場したことにより、ファインチューニング最大の壁である「データ準備の手間」が激減できそうです。他にも「蒸留」や「強化学習」の話もあり、AI開発のレベルを一段階上げたい方は必見なセッションです。

https://ignite.microsoft.com/en-US/sessions/BRK188

### Let your agentic apps talk with Azure Speech（BRK 198）
「[Speech MCP Servers](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/tools/azure-ai-speech?view=foundry)」や「Photo Avatar」（Public Preview）など新機能の紹介もあり、音声を活用した生成AIのビジネス適用のイメージがわいてくるセッションとなっており、必見です。

https://ignite.microsoft.com/en-US/sessions/BRK198

### Foundry Control Plane: Managing AI agents at scale（BRK202）
様々な技術進展により、"AIエージェントを作ること自体"は非常に簡単になりました。これからは、作ったエージェントが暴走しないよう「管理・統制（ガバナンス）」することが企業の競争力になります。そこで、本セッションでは、「どうやって数千・数万のエージェントを安全に運用するか？」というスケーリング（規模拡大）のフェーズにおける論点への回答として、Microsoft Foundryから新たな機能として「Foundry Control Plane」の紹介があり、この機能は一言で言えば、「社内外のあらゆるAIエージェントを一元管理・監視・制御するための司令塔」的な機能です。AIエージェントのガバナンスについて、興味関心（+悩み）のある方はぜひ本セッション、ご覧ください！何かヒントがあるかもしれません。

https://ignite.microsoft.com/en-US/sessions/BRK202

## [Kazuki Ota](https://zenn.dev/okazuki)
生成 AI が出てから 3 年で、ようやく Agent を作るための方法が確立されてきたように感じます。Ignite では Microsoft Foundry Agent Service が開発者向けの Agent 作成ツールで Copilot Studio が市民開発者向けの Agent 作成ツール、Microsoft Agent Framework が開発者が最高の自由度とカスタマイズ性を備えた Agent 開発のフレームワークという位置付けをしてきたように感じています。私はコードを書きたい人なので、今見た中で一番 Microsoft Agent Framework について取り上げられているセッションを紹介します！

### AI powered automation & multi-agent orchestration in Microsoft Foundry（BRK 197）
Microsoft Agent Framework を中心にワークフローの構築方法やワークフローの可視化を行う DevUI、Semantic Kernelや AutoGen からの移行ツール（GitHub Copilot）、Purview、hosted agent など一通り開発者向けの機能が紹介されています。

https://ignite.microsoft.com/en-US/sessions/BRK197?source=sessions


## [Junpei Tsuchida](https://zenn.dev/07jp27)
Ignite 2025の新出キーワードは「IQ」でした。IQは知能指数の他にも英語ネイティブ的には「頭のキレ」みたいな意味があるそうです。

### Make smarter model choices: Anthropic, OpenAI & more on Microsoft Foundry（BRK195）
今回のイベントで最もインパクトがあるといっても過言ではないAzure上でのClaudeモデルのリリース！
しかしモデルが溢れてくる昨今において、どのモデルを選択すればよいのか迷う方も多いはず。
本セッションでは、Anthropic Claudeモデルの特徴や、他の主要モデルとの比較を通じて、ユースケースに最適なモデル選択のポイントを解説しています。
モデル選択に悩んでいる方は必見のセッションです！

また、このセッションはMicrosoft Foundry全体の概要を掴むのにも最適なセッションとなっていますので、Foundryに興味がある方はぜひご覧ください！
https://x.com/07JP27/status/1992814251279634901

https://ignite.microsoft.com/en-US/sessions/BRK195



## 全セッションアーカイブ一覧
全てのセッションは以下のリンクからアーカイブ動画を視聴できます。
ご興味のあるセッションがありましたら、ぜひ視聴してみてください！

https://ignite.microsoft.com/en-US/sessions

## Build 2025 注目セッションまとめ
前回の Build 2025 注目セッションまとめもぜひご覧ください！

https://zenn.dev/microsoft/articles/ms-pub-build-2025