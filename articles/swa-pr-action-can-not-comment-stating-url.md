---
title: "Azure Static Web AppsのPull RequestワークフローでプレビューURLがコメントされない"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "github", "staticwebapps"]
published: true
publication_name: "microsoft"
---

## 事象
Azure Static Web Appsは紐づけたリポジトリにActionsのワークフローファイルを自動で作成されます。
そのワークフローファイルはPull Requestを作成するとPRのコードで自動的にAzureにプレビュー用の環境を作成してくれます。
ドキュメントによると、プレビュー用の環境が作成されると、PRのコメントにURLが自動でコメントされるはずなのですが、私の環境ではコメントされない事象が発生しました。

本来であれば以下のようなコメントがPRに自動でコメントされるはずです。

![](/images/swa-pr-action-can-not-comment-stating-url/comment.png)
https://learn.microsoft.com/ja-jp/azure/static-web-apps/review-publish-pull-requests#review-changes


## 原因
原因はGitリポジトリの設定のワークフローの実行権限に書き込みが許可されていないためです。

## 設定方法
対象のリポジトリの「Settings」→「Actions」→「General」にアクセスします。
![](/images/swa-pr-action-can-not-comment-stating-url/1.png)

「Workflow permissions」の項目で「Read and write permissions」を選択し、「Save」をクリックします。
![](/images/swa-pr-action-can-not-comment-stating-url/2.png)


設定を保存後にPRのコードを更新したり、新規にPRを作成したりすると、PRのコメントにプレビューURLがコメントされるようになります。
![](/images/swa-pr-action-can-not-comment-stating-url/3.png)
