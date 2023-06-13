---
title: "Bicepファイルのパラメーターファイルを自動生成する"
emoji: "👇"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['bicep', 'azure', 'iac']
published: true
publication_name: "microsoft"
---

:::message
2023年6月12日にBicepの新しいパラメーター形式である.bicepparamが正式リリースされました。今後はそちらがスタンダードになっていくと考えられます。
引き続きJSON形式のパラメーターファイルを使用する必要がある場合は、この記事をご参照ください。
:::

.bicepparamの記事はコチラ！
https://zenn.dev/microsoft/articles/bicep-bicepparam-file

-------------------

「Bicep使ってもパラメーターファイルはARMと同じく書きづらいJSONだしな～」そう思ってた時代が私にもありました。
# それ、自動生成できますよ。

はい、ドーン
```
az bicep generate-params --file main.bicep
```

https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/bicep-cli#generate-params

# 注意点
- 生成されるのはBicepファイルで規定値が構成されていないパラメーターのみです。＝ 規定値のあるパラメーターは生成してくれません。
- 生成されるファイル名は {bicepファイル名}.parameters.json です。すでに同じ名前の parameters.json ファイルがある場合は更新します。

# やってみた
こんなBicepファイルを用意します。
パラメーターは２つ。１つは規定値なし、１つは規定値ありです。
![](/images/bicep-generate-params/bicep.png)

```
az bicep generate-params --file main.bicep
```

注意点通り規定値無しの`storageNamePrefix`だけが生成されました。あとは値を入れるだけですね。
![](/images/bicep-generate-params/params.png)



CLIのドキュメントをざっと流せばリストアップされてるのですが、私の周りで話題になったことはあんまりない気がします(知られてない？)。便利ですね。