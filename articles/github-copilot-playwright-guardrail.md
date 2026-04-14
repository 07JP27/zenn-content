---
title: "Agentic Web Accessのガードレール戦略 / GitHub Copilot x Playwright CLI"
emoji: "🕸️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['githubcopilot', 'playwright']
published: false
publication_name: "microsoft"
---

## GitHub Copilot x Playwright CLI による強力なAgentic Web Access
もともとPlaywrightはWebアプリのE2Eテストフレームワークであり、事前にアクセスフローをコードで書いておく必要がありました。そこに[Playwright MCP](https://github.com/microsoft/playwright-mcp)や[Playwright CLI](https://github.com/microsoft/playwright-cli)が登場したことで、その場でアドホックにブラウザの自動操作ができるようになりました。さらにそれをAIエージェントから操作することで、Webブラウザへのアクセスをエージェンティック（自律的かつ反復的）に行えるようになっています。

:::message
Playwright MCPとPlaywright CLIはどちらもAIエージェントにブラウザ操作を提供しますが、役割が異なります。CLIはSkillsと組み合わせることでトークン効率が良く、コーディングエージェントでは主流になりつつあります。一方MCPは持続的なブラウザコンテキストを活かした探索的な自動化や、長時間の自律ワークフローに向いています。詳しくは[Playwright MCP vs Playwright CLI](https://github.com/microsoft/playwright-mcp#playwright-mcp-vs-playwright-cli)を参照してください。
:::

この手法の出現によりPlaywrightはE2Eテストだけでなく、さまざまな用途に利用できるようになりました。

- **手順書作成**：目的を伝えてAIが試行錯誤した結果、成功パターンのみを手順書としてマークダウン形式で作成
- **申請代行**：入力データを与えるとAIがWebフォーム入力を代行してSubmitする
- **多言語サイト差分検出**：多言語サイトにそれぞれアクセスし、言語以外の差分がないかを確認

E2Eテスト以外のGitHub Copilot x Playwright CLIシナリオ集を作成したので、こちらもぜひ合わせてご覧ください。
https://github.com/07JP27/GitHubCopilot-PlaywrightCli-Demo

## 自律的だからこそ制御が必要
エージェンティックにWebアクセスを行うということは、AIが自律的に判断してサイトにアクセスしたり、ページの中身を読み取ったりするということです。業務上アクセスしてほしくないサイトなどをブロックする必要があります。
この記事では、このシナリオで取りうる防御策（ガードレール）について見ていきます。

## アクセスパターンを洗い出す
![](/images/github-copilot-playwright-guardrail/access-pattern.png)

まず防御を考える前に、Agentic Web Accessにおけるアクセスパターンを整理します。Playwrightによるブラウザアクセスも、CopilotのfetchツールによるWebアクセスも、いずれもローカルマシンから対象サイトへ直接リクエストが送信されます。ただし、ブラウザプロセス（Playwright）やElectronプロセス（VS Code）からの通信と、Node.jsプロセス（Copilot CLI）からの通信では、OS/ネットワークレベルの防御が効くかどうかが異なります。この違いは後述する防御レイヤーの有効性に大きく影響します。

また、防御をレイヤーで考える必要があるのは、アクセスの起点がPlaywrightだけとは限らないからです。Playwrightの結果をもとにCopilotがさらにWebアクセスをする可能性がありますし、Copilotから提示されたリンクをユーザーがクリックしてアクセスする可能性もあります。

## 多層防御を考える
代表的な機能やサービスを使った防御策の全体像は以下のようになります。
![](/images/github-copilot-playwright-guardrail/block-pattern.png)

### Playwright CLI/MCP
#### Origin制御
Playwright CLI/MCPには、環境変数によるオリジン制御の仕組みが用意されています。`@playwright/mcp` と `playwright-cli` は内部で同じコードを共有しているため、Origin制御の仕組みも共通です。ここではまとめて説明します。

オリジン制御にはブロックリストと許可リストの2種類があり、セミコロン区切りで複数オリジンを指定できます。

| 環境変数 | 動作 |
|---|---|
| `PLAYWRIGHT_MCP_BLOCKED_ORIGINS` | ブロックリスト：指定したオリジンだけをブロックし、それ以外は許可 |
| `PLAYWRIGHT_MCP_ALLOWED_ORIGINS` | 許可リスト：指定したオリジンだけを許可し、それ以外はすべてブロック |

両方を同時に指定した場合、ブロックリストが先に評価されます（両方に含まれるオリジンはブロック）。

ブロックされたオリジンにアクセスすると `net::ERR_BLOCKED_BY_CLIENT` エラーが発生し、以下のような画面が表示されます。
![](/images/github-copilot-playwright-guardrail/playwright.png)

ブロックと許可を同時に設定できるため、「社内の `*.example.com` は全て許可するが、本番環境だけは誤操作防止のためブロックする」といった設定も可能です。

```bash
export PLAYWRIGHT_MCP_ALLOWED_ORIGINS="https://*.example.com"
export PLAYWRIGHT_MCP_BLOCKED_ORIGINS="https://prod.example.com"
```

詳細は[Playwright MCPのConfigurationドキュメント](https://github.com/microsoft/playwright-mcp#configuration)を参照してください。

:::message alert
このOrigin制御はPlaywrightのアプリケーションレベルのルートインターセプションで実装されているため、悪意あるページのJavaScript等による迂回を完全にはブロックできません。また、許可されたオリジンからブロック対象のオリジンへのリダイレクトはブロックされません。Playwrightのブラウザアクセスにはネットワーク/OSレベルの防御（MDEやEntra Internet Access）も有効ですが、CopilotのfetchツールはNode.jsプロセスから実行されるため、macOS版MDEの検査対象外となる点に注意が必要です。ブラウザアクセスとfetchアクセスの両方をカバーするには、preToolUseフック等のCopilotレイヤーの防御も組み合わせることが重要です。
:::

### GitHub Copilot
#### URLパーミッション（Trusted Domains）
VS Code上のGitHub Copilotエージェントモードでは、Webコンテンツの取得時に2段階の承認フローが設けられています。まずアクセス先のドメインを信頼するかどうかを確認され、次に取得したコンテンツをモデルに渡す前にレビューの機会が提供されます。この仕組みはVS Codeの[Trusted Domains](https://code.visualstudio.com/docs/editing/editingevolved#__outgoing-link-protection)リストと統合されており、一度信頼したドメインは以降の承認が不要になります。

一方、[Copilot CLI](https://code.visualstudio.com/docs/copilot/agents/copilot-cli)はVS Codeのプロセス外で動作するため、このTrusted Domainsによる承認フローは適用されません。また、Copilot CLI自体にもドメインフィルタの仕組みはありません。さらに、CLIのfetchはNode.jsプロセスから実行されるため、macOS上のMDE Network Protectionの検査対象外でもあります。Copilot CLIにおいては、後述するpreToolUseフックや`--deny-url`フラグが主要な防御手段となります。

詳しくは[VS Codeのセキュリティドキュメント](https://code.visualstudio.com/docs/copilot/security)を参照してください。

:::message
この記事ではローカルでの実行を前提にしています。Cloud Agent（コーディングエージェント）はGitHubのクラウドインフラ上で動作するため、ここで紹介したローカルの防御レイヤー（MDE、Entra Internet Access等）は適用されません。Cloud Agent向けには専用のファイアウォール設定が重要です。詳しくは[Cloud Agentのファイアウォール設定](https://docs.github.com/ja/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/cloud-agent/customize-the-agent-firewall)を参照してください。
:::

#### インストラクション
確実性はないものの、他のアクセス防御がアクセス発生後にリアクティブに働くのに対し、インストラクションはアクセスが発生する前にプロアクティブに働く可能性があります。例えば「`https://prod.example.com` には絶対にアクセスしないでください」といった指示をカスタムインストラクションに記載しておくことで、AIがそもそもアクセスを試みないようにできます。確実な制御ではありませんが、重要なガードレールの一つです。

#### preToolUseフック
インストラクションが「お願い」ベースであるのに対し、[preToolUseフック](https://code.visualstudio.com/docs/copilot/customization/hooks)はツール実行をプログラム的にブロックできる仕組みです。preToolUseはエージェントがツールを実行する直前に発火し、フックスクリプトがJSON形式で `{"permissionDecision":"deny"}` を返すとそのツール呼び出しがブロックされます。

例えば、`fetch`ツールやシェルの`curl`/`wget`コマンドに含まれるURLをブロックリストと照合するスクリプトを書くことで、特定ドメインへのアクセスをリポジトリ単位でプログラム的に制御できます。

`.github/hooks/url-policy.json`:
```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      {
        "type": "command",
        "bash": ".github/hooks/check-url.sh",
        "timeoutSec": 10
      }
    ]
  }
}
```

`.github/hooks/check-url.sh`（簡易例）:
```bash
#!/bin/bash
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.toolName')
TOOL_ARGS=$(echo "$INPUT" | jq -r '.toolArgs')

# fetchツールのURLを検査
if [ "$TOOL_NAME" = "fetch" ]; then
  URL=$(echo "$TOOL_ARGS" | jq -r '.url // empty')
  if echo "$URL" | grep -qE "(evil\.com|prod\.example\.com)"; then
    echo '{"permissionDecision":"deny","reason":"Blocked by org policy"}'
    exit 0
  fi
fi

# shellコマンド内のcurl/wgetも検査
if [ "$TOOL_NAME" = "shell" ]; then
  CMD=$(echo "$TOOL_ARGS" | jq -r '.command // empty')
  if echo "$CMD" | grep -qE "curl|wget" && \
     echo "$CMD" | grep -qE "(evil\.com|prod\.example\.com)"; then
    echo '{"permissionDecision":"deny","reason":"Blocked by org policy"}'
    exit 0
  fi
fi
```

`.github/hooks/` にコミットしておけば、VS CodeのエージェントモードとCopilot CLIの両方で動作します。

:::message alert
preToolUseフックには以下の限界があります。

- **ローカルで削除可能**：ユーザーがリポジトリをcloneした後、`.github/hooks/` をローカルで削除すればフックは読み込まれません。Gitのpre-commit hookと同じ弱点です
- **シェルコマンドの解析はヒューリスティック**：`shell(curl ...)` は検出できますが、`shell(python -c "import requests; ...")` のような間接的なアクセスはスクリプトの解析能力に依存します
- **フェイルオープン**：フックがクラッシュまたはタイムアウトした場合はスキップされ、エージェントの実行はブロックされません。フックの実装バグには注意が必要です
:::

### Microsoft Defender for Endpoint Web Protection
[Microsoft Defender for Endpoint（MDE）のWeb Protection](https://learn.microsoft.com/ja-jp/defender-endpoint/web-protection-overview)は、デバイス単位でWebアクセスを制御する機能です。Web脅威保護、Webコンテンツフィルタリング、カスタムインジケーターの3つの機能で構成されています。

- **Web脅威保護**：フィッシングサイトやマルウェアサイトへのアクセスを自動でブロック
- **Webコンテンツフィルタリング**：カテゴリに基づいてWebサイトへのアクセスを制御
- **カスタムインジケーター**：特定のURL/IPアドレスに対してブロック・許可・警告のアクションを設定

ただし、MDE Network Protectionの検査範囲はプロセスの種類によって異なります。実際に検証したところ、以下の結果が得られました。

| アクセスパターン | MDE Network Protection (macOS) | preToolUseフック |
|---|---|---|
| VS Code Agent Mode (fetch) | ✅ 有効（Electronプロセス） | ✅ 有効 |
| Copilot CLI (fetch) | ❌ すり抜ける（Node.jsプロセス） | ✅ 有効 |
| Playwright CLI/MCP（ブラウザ） | ✅ 有効（ブラウザプロセス） | ✅ 有効 |

macOS版のMDE Network Protectionはネットワーク拡張（Network Extension）ベースで動作しており、検査対象はブラウザプロセスに限定されています。VS CodeはElectron（Chromiumベース）であるためブラウザプロセスとして認識・検査されますが、Copilot CLIのNode.jsプロセスは検査対象外となります。

:::message
Windows版ではWFP（Windows Filtering Platform）ベースで動作が異なるため、検査範囲が異なる可能性があります。エンタープライズ環境では実際に検証することをお勧めします。
:::

ブロックされるとOSにこのような通知が届きます。
![](/images/github-copilot-playwright-guardrail/mde.png)

### Microsoft Entra Internet Access
[Microsoft Entra Internet Access](https://learn.microsoft.com/ja-jp/entra/global-secure-access/concept-internet-access)は、ID（ユーザーID）を中心としたSecure Web Gateway（SWG）ソリューションです。Webコンテンツフィルタリングにより、Webカテゴリや完全修飾ドメイン名（FQDN）に対するきめ細かなアクセス制御が可能です。

MDEのWeb Protectionがデバイス単位の制御であるのに対し、Entra Internet Accessは条件付きアクセスポリシーと連携することで、ユーザーやグループ単位でのポリシー適用が可能です。テナント全体で統一されたWebアクセスポリシーを適用したい場合に適しています。

## VS Code Agent Mode vs Copilot CLI：防御レイヤーの格差
ここまで各防御レイヤーを紹介してきましたが、実はVS Code Agent ModeとCopilot CLIとではガードレールの厚みに大きな差があります。これは「CLIが危険」ということではなく、**防御の設計思想の違い**です。

VS Code Agent Modeは多層防御が機能します。Trusted Domainsによる2段階承認、MDE Network Protection（Electronプロセスとして検査対象）、preToolUseフック、インストラクションと、複数のレイヤーが重なっています。

一方、Copilot CLIは利便性と自律性を重視した設計であり、防御はユーザーの設定と運用に大きく依存します。Copilot CLIにおける各防御手段の状態を整理すると以下のとおりです。

| 防御手段 | Copilot CLIでの状態 |
|---|---|
| MDE Network Protection | ❌ Node.jsプロセスは検査対象外（macOS実証済み） |
| Trusted Domains / 2段階URL承認 | ❌ VS Code固有機能、CLIには存在しない |
| preToolUseフック | ⚠️ フェイルオープン設計（異常終了・タイムアウト時はスキップ）。Anthropic BYOMでは[発火しない](https://github.com/github/copilot-sdk/issues/893) |
| URL権限（`allowed_urls`/`denied_urls`） | ⚠️ ローカルconfig。`--allow-all-urls`で無効化可能 |
| `--deny-url`フラグ | ✅ 有効だがユーザーが起動時に指定する前提 |

エンタープライズ環境でCopilot CLIの`web_fetch`に対する防御を強化したい場合は、以下のアプローチを検討してください。

- **ネットワークレベルでの制御**：ファイアウォールやプロキシによるホワイトリスト方式でアクセス先を制限する
- **ツール自体の無効化**：`--deny-tool='web_fetch'`でfetchツールそのものを無効化する
- **preToolUseフックの導入と監視**：フェイルオープンである点を認識した上で、フックの正常動作を監視する運用を組み合わせる

## まとめ
この記事ではGitHub Copilot x Playwright CLI/MCPによるAgentic Web Accessにおけるガードレール戦略を紹介しました。各レイヤーの防御策を一覧にまとめます。

| レイヤー | 防御策 | 設定単位 | 設定する人 |
|---|---|---|---|
| Playwright CLI/MCP | Origin制御（環境変数） | 実行環境単位 | 開発者 |
| GitHub Copilot | URLパーミッション | 実行環境単位 | 開発者 |
| GitHub Copilot | インストラクション | リポジトリ単位/端末単位 | リポジトリ管理者/開発者 |
| GitHub Copilot | preToolUseフック | リポジトリ単位 | リポジトリ管理者/開発者 |
| Microsoft Defender for Endpoint | Web Protection | 端末単位 | テナント管理者/開発者 |
| Microsoft Entra | Internet Access | テナント単位 | テナント管理者 |

一つのレイヤーだけに頼るのではなく、必要な防御策を適切に組み合わせて多層防御を構築することが重要です。特にCopilot CLIを利用する環境では、VS Code Agent Modeとは防御レイヤーの厚みが異なる点を認識し、ネットワークレベルの制御やツール無効化など追加の対策を検討してください。Agentic Web Accessの利便性を活かしつつ、安全に運用するための参考になれば幸いです。

※ MDE Web Protectionのシナリオ例：macOSではNode.jsプロセス（Copilot CLI等）に対して無効。Windows版では検査範囲が異なる可能性あり
