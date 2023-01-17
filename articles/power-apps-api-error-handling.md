---
title: "Power AppsのAPI呼び出しでエラーハンドリングする"
emoji: "🚗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [powerapps]
published: true
publication_name: "microsoft"
---

:::message
この記事の内容は執筆時点で「実験段階」の機能を使用します。
:::

Power Appsでカスタムコネクタとして読み込んだ外部のAPIを使用した際にエラーレスポンスが返ってきた場合にハンドリングする方法です。

# 前提
Power AppsにOpen API定義ファイルを読み込んでカスタムコネクタを作成しておきます。
具体的な方法は以下の記事が参考になります。
https://zenn.dev/okazuki/articles/azure-function-powerapps

# 手順
## 1.実験段階の機能をON
アプリの「ファイル」→「設定」→「近日公開の機能」→「実験段階」から「数式レベルのエラー管理」をONにします。
![](https://storage.googleapis.com/zenn-user-upload/75c31a25afb9235b38f0c618.png)


## 2.カスタムコネクタの読み込み
「データ」→「データの追加」をクリックして前提で作成したカスタムコネクタを読み込みます。
![](https://storage.googleapis.com/zenn-user-upload/73a76fc1ec14c452250481a5.png)

## 3.API呼び出し処理の実装
IfError関数を使用します。
```
IfError(
    AzureFunctionsOpenAPIExtension.Debug(200),　// エラーを評価される処理
    UpdateContext({message:"error"}),　// エラーの場合の処理
    UpdateContext({message:"success"})　//　正常に終了したときの処理
);
```

今回はテストとして、リクエストに欲しいレスポンスコードを渡すとそのレスポンスコードを返してくれるAPIを使用しています。
![](https://storage.googleapis.com/zenn-user-upload/a71ee790e190b1a06035ba3a.gif)


これでエラー時のハンドリングが実装できました。

# 参考
Power Appsには今回使用したIfError関数の他にもエラーをハンドリングできる関数が存在します。また、IfError関数にも様々なオプションがあり、より柔軟な処理を書くことができます。
https://docs.microsoft.com/ja-jp/powerapps/maker/canvas-apps/functions/function-iferror