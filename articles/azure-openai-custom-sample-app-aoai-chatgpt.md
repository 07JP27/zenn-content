---
title: "Azure OpenAI Studioからデプロイできるチャットアプリをコーディングレスでカスタマイズする"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'openai', 'chatgpt']
published: true
publication_name: "microsoft"
---
## Azure OpenAI Studioからデプロイできるチャットアプリ

Azure OpenAI StudioやAzure AI Studioからデプロイできるチャットアプリをご存知でしょうか。
それぞれのStudioからデプロイをクリックすると、Azure App Serviceにデプロイされて、すぐにチャットアプリを使うことができます。

![Azure OpenAI ポータルからのデプロイ](/images/azure-openai-custom-sample-app-aoai-chatgpt/aoai_studio.png)
*Azure OpenAI ポータルからのデプロイ*

![Azure AI Studioからのデプロイ](/images/azure-openai-custom-sample-app-aoai-chatgpt/ai_studio.png)
*Azure AI Studioからのデプロイ*

デプロイされたアプリは以下のようなUIになっています。

![](/images/azure-openai-custom-sample-app-aoai-chatgpt/original.png)

非常に簡単にデプロイできる一方でこのまま企業内で使うには少しUIカスタマイズが必要かもしれません。しかし、実はこのアプリは**コードを変更することなく**デプロイ後にUIがカスタマイズできるようになっています。

## カスタマイズすると・・・
コードを変えることなくこんな感じのUIにカスタマイズできます。写真の部分は自社のアイコンなどにしてオリジナリティを出せます。
![](/images/azure-openai-custom-sample-app-aoai-chatgpt/customized.png)

## カスタマイズの方法
- [Azure portal](https://portal.azure.com)にログインして、各StudioからデプロイされたApp Serviceを見つけます。デプロイする際に設定した「名前」をAzure portal上部の青色ヘッダーにある検索ウインドウに入力して検索すると見つけやすいです。
- App Serviceの左側のメニューから「環境変数」を選択します。
- 環境変数の「追加」をクリックして名前に「UI_CHAT_TITLE」、値に「チャットを始めよう」と入力して「適用」をクリックします。
![](/images/azure-openai-custom-sample-app-aoai-chatgpt/1.png)
![](/images/azure-openai-custom-sample-app-aoai-chatgpt/2.png)
- 環境変数に変更があれば環境変数一覧の下部の「適用」がクリックできるようになっているので「適用」をクリックします。**App Serviceが再起動します。本番運用への適用は慎重に行ってください。**
![](/images/azure-openai-custom-sample-app-aoai-chatgpt/3.png)
- しばらくしてからアプリにアクセスしてタイトルの表示が設定した値に変わったことを確認します。
![](/images/azure-openai-custom-sample-app-aoai-chatgpt/4.png)


## カスタマイズできる項目
ここまで読んでいただいたらわかるように環境変数の名前と値を変えるだけでカスタマイズできました。
ただし、環境変数によってカスタマイズできる部分は限られています。以下にカスタマイズできる項目を示します。

| 環境変数名 | 説明 |
| --- | --- |
| UI_TITLE | 左上のヘッダータイトル。 |
| UI_LOGO | 左上のロゴ。画像URLを指定するのでWebサイトの画像URLやBlob Storageに画像を保存してそのURLを記載する。 |
| UI_CHAT_TITLE | チャットが開始されていない時に表示されるタイトル。 |
| UI_CHAT_LOGO | チャットが開始されていない時に表示されるロゴ。UI_LOGOと同様にURLを指定する。 |
| UI_CHAT_DESCRIPTION | チャットが開始されていない時に表示されるタイトルの下に記載される説明文。 |
| UI_FAVICON | ファビコン。UI_LOGOと同様にURLを指定する。 |
| UI_SHOW_SHARE_BUTTON | 右上のShareボタンを非表示にするかどうかをTrue/Falseで指定する。何も設定しないとTrueになる。 |

## まとめ
「実はここの部分もカスタムできるんだぜ」というご紹介でした。
このサンプルアプリにはOn your dataや認証、Cosmos DBを使った会話履歴の保存やフィードバックの収集などが実装されているので、「ここが変えられるならこれでええやん」となるケースも多いかもしれません。

## 余談1 - システムプロンプトの設定
`AZURE_OPENAI_SYSTEM_MESSAGE`の環境変数を設定することでシステムメッセージ(メタプロンプト)を変更できます。これでチャットのキャラクターや役割などを変更できます。
その他にもUI以外で環境変数を変更してカスタマイズできる項目はGitHubに掲載されています。
https://github.com/microsoft/sample-app-aoai-chatGPT?tab=readme-ov-file#environment-variables

## 余談2 - コーディングを伴うカスタマイズ
ポータルのデプロイボタンからデプロイされるコードはオープンソースで公開されているの、ローカルで動かしてみたり、さらに深いコーディングを伴うカスタマイズも可能です。
ローカルデバッグの方法やローカルからAzureへのデプロイ方法はリポジトリのREADMEにステップバイステップで記載してあるのでより詳細なカスタマイズを行いたい読者はこの方法を試してみてください。

https://github.com/microsoft/sample-app-aoai-chatGPT

## 参考
https://learn.microsoft.com/ja-jp/azure/ai-services/openai/how-to/use-web-app