---
title: "GitHub Copilotのユーザー入力待ちを通知する物理デバイスを作ってみた"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['githubcopilot']
published: true
publication_name: "microsoft"
---

この記事はZennの「[GitHub Copilot活用選手権](https://zenn.dev/contests/github-2026-spring)」を盛り上げるため~~のネタ枠として~~書いています。
ぜひ皆さんのGitHub Copilotの活用アイデアも投稿してみてください！

---

GitHub Copilotでユーザーの入力待ち（Ask User）の状態を検知して、LEDとCopilot君が物理的に通知する装置を作ってみました!
応えてくれるユーザーを探しているような動作をイメージしています。
この記事では、この装置を作るに至ったきっかけや、装置の技術的な仕組み、今後の展望などについて紹介していきます。

![](/images/github-copilot-physical-notifier/demo.gif =100%x)


3Dモデルやファームウェア、ビルドガイドはGitHubでMITライセンスの下、オープンソースとして公開しています。この記事を読んで興味が湧いたら、ぜひリポジトリをチェックしてみてください！
https://github.com/07JP27/GitHub-Copilot-Physical-Notifier




## きっかけ
### GitHub Copilotは飛躍的に進化した！ しかし・・・
GitHub CopilotはAutopilot機能の追加など、より自律的に動作するように進化を続けています。現在では、ある程度放置していても質の高いコードを生成してくれるようになりました。
しかし自律的になったがゆえに、意気揚々と指示を出して休憩したものの、戻ってきたらユーザーの入力待ちで全然処理が進んでいなかった…というのはAIコーディングあるあるではないでしょうか。特にGitHub Copilot CLIはバックグラウンドで並列実行することが多いためこの状況に陥りやすいです。

これをなるべくロス無く気づくための通知の仕組みが欲しいところですが、巷で行われている音を鳴らす方法は、音量をゼロにしていたり1回鳴っただけでは気づけないこともあります。

### AI Tour Tokyoでの出会い
そんな中、今回の装置を思いついたのが2026年3月に開催されたMicrosoft AI Tour TokyoのGitHubブースで動くCopilot君を見たときでした。（かわいい）

https://x.com/07JP27/status/2036225854112735450?s=20

これをミニチュア化したもので通知できるデバイスがあれば可愛い&面白いのではないかと思ったのがきっかけで製作をはじめ、ようやく完成しました。


## Copilot君の首振りを支える技術
ここからは少し技術的な部分を紹介します。

### 全体構成
全体的な仕組みとしては非常にシンプルで、PCとArduinoをUSBで接続し、PC側でGitHub Copilotのユーザーの入力待ち状態を検知したらArduinoに信号を送り、Arduinoが信号を受け取ったらサーボモーターとLEDを動かすという流れになります。
![](/images/github-copilot-physical-notifier/structure.png)


### ハードウェア
#### 部品選定とケース設計
今回使用した部品は以下の通りです。(ケーブル類など細かい部品を除く)
| # | 部品 |
|---|------|
| 1 | Arduino Nano （互換でも可） |
| 2 | SG90 マイクロサーボ |
| 3 | 5mm砲弾型LED |
| 4 | 抵抗 |
| 5 | タクトスイッチ |



これらが収まるように、なるべく小型化を意識してCADでケースを設計しました。
![](/images/github-copilot-physical-notifier/cad.png)
ケースの3Dモデルも[GitHubリポジトリ](https://github.com/07JP27/GitHub-Copilot-Physical-Notifier/tree/main/models)で公開しています。

部品点数が少ないため、結線図も以下のようにシンプルです。
![](/images/github-copilot-physical-notifier/circuit-diagram.png)


#### octoprints
これがこの装置の象徴と言っても過言ではありません!
octoprintsはGitHubの3大？キャラクターである Mona / Copilot / Duckyの3Dモデルを提供しているオープンソースプロジェクトです。
![](/images/github-copilot-physical-notifier/octoprints.png)

https://github.com/martinwoodward/octoprints

個人の方のプロジェクトですが、`The mascots are used with permission of GitHub.（マスコットキャラクターはGitHubの許可を得て使用しています。）`とされており、CC BY-NC 4.0ライセンスの元、非商用であれば誰でも自由に使えます。


### GitHub Copilot Hooks
[GitHub Copilot Hooks](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-hooks)は、Copilotエージェントのワークフローの特定のタイミング（セッション開始・終了、ツール実行前後など）でカスタムシェルコマンドを実行できる仕組みです。`~/.copilot/hooks/`にJSON形式でフック定義を配置することで利用できます。今回はこのフックを使ってCopilotの状態変化を検知しています。

hooksは現在も機能拡張を続けており、直近では、Copilotがユーザーの入力・確認を求めているときに発火する`Notification hook`が、2026年1月28日からissueで議論され、2026年4月7日のv1.0.18で実装されました。
https://github.com/github/copilot-cli/issues/1157

今回は`preToolUse hook`と`postToolUse hook`を利用して、Copilotがユーザーの入力待ちになったタイミングを検知して動作開始、ユーザーが入力を完了したら動作停止するようにしています。具体的には、フックに渡されるJSONの`toolName`が`ask_user`であるかをフィルタすることで、ユーザーの入力待ち（Ask User）のみを判別しています。

#### 既知の問題
現在、権限確認ダイアログの検知には対応できていません。`Notification hook`を使う方法も検討しましたが、すでに権限を許可している場合でもすべての`Notification hook`が発火してしまうため、ユーザーの入力待ち状態だけを正確に検知できないためです。

こういう権限確認を検知しようとすると
![](/images/github-copilot-physical-notifier/permission.png)
すでに許可していても、この緑点すべてで発火してしまう。
![](/images/github-copilot-physical-notifier/actions.png)

この問題については以下のissueを作成したので今後解消されることを期待しています。
https://github.com/github/copilot-cli/issues/2586


## 今後の展望
今回は実証実験もかねてミニマルな機能と部品で制作しましたが、今後は以下のような機能追加や改良をしていきたいと考えています。
- フルカラーLEDやディスプレイなどを使ったリッチな表現
- １軸の首振りだけではなく3次元的な動きの実装
- シリアル通信ではなくWi-FiやBluetoothを使ったワイヤレス化
- Copilotの状態に応じた多様なアクション（例：コード生成中は頷きながら考えるなど）


## まとめ
というわけでGitHubを最大限活用しようとした結果、物理空間に飛び出してしまった記事でした。
ここで紹介したデバイスは誰でも再現できるようにGitHubでオープンソースとして公開しています。ビルドガイドも丁寧に書いたので、この記事で興味が湧いたら作ってみてください！

https://github.com/07JP27/GitHub-Copilot-Physical-Notifier

それでは良いGitHub Copilotライフを！



### 余談：対抗現る
制作も終盤に差し掛かった頃にClaude Codeのキャラクターで同じような装置を作っている方がいることを知りました。
https://x.com/07JP27/status/2041375592336199801?s=20