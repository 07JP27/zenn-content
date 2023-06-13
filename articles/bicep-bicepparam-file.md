---
title: "Azure Bicep に bicepparamファイル 爆 誕 "
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["bicep", "azure"]
published: true
publication_name: "microsoft"
---

TL;DR
Bicep 0.18.4から .bicepparamという拡張子のファイルでパラメーターファイルがサポートされました。
この記事は[BicepのGitHubリポジトリのリリース情報](https://github.com/Azure/bicep/releases/tag/v0.18.4)を元に執筆しています。

# はじめに - いままでの課題 -
Azure純正のInfrasturcture as Code(IaC)ツールであるBicepは、Azure Resource Manager(ARM)テンプレートをより簡潔に記述できるようにするために開発されました。
Bicepの登場により、JSON形式で記述する必要があったARMテンプレートの問題がいくつか解消されました。


#### Bicepの特徴
- プログラミング言語ライクで、より人間にわかりやすい記述に
- モジュールを使用したリソース再利用性の向上
- コードにコメントが書ける！（ARMテンプレートはJSON形式なのでコメントが書けなかった）

しかし、BicepによりAzureのIaCは大きく進化しましたが、Bicepで定義したテンプレートに与える変数のパラメーターファイルの記述にはJSON形式を使用する必要がありました。

# .bicepparamファイルの登場
この度、現地時間の2023年6月12日にリリースされたBicep 0.18.4から .bicepparamという拡張子のファイルでパラメーターファイルを作成できる機能が正式サポートされました！
これにより、Bicepで定義したテンプレートに与える変数のパラメーターファイルの記述にJSON形式を使用する必要がなくなりました。

#### .bicepparamファイルの特徴
- 簡潔な記述方式
- コメントが書ける！
- テンプレートファイルに対するコード補完(厳密にはVS CodeのBicep拡張機能の機能)


# やってみる
## 準備
前述の通りBicepのバージョンが0.18.4以上である必要があります。 古いバージョンの場合は```az bicep update```でアップデートしてください。真っさらな状態から新規インストールする方は[こちら](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/install)を参考にしてください。
Bicepファイルを記述するために、IDEとして、VS Code、さらに[Bicep拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep)をインストールします。これはBicep開発のデファクトスタンダートとなっています。Bicep拡張機能もv0.18.4から.bicepparamファイルのサポートがされています。

## 適当にBicepファイルを作る
適当にこんな感じのストレージを１個デプロイするためのBicepファイルを作ります。
``` bicep
@minLength(3)
@maxLength(13)
@description('The storage name will be {this value}storage1128')
param storageNamePrefix string

param location string = resourceGroup().location

var storageName = '${storageNamePrefix}storage1128'

resource exampleStorage 'Microsoft.Storage/storageAccounts@2021-06-01' = {
    name: storageName
    location: location
    sku: {
        name: 'Standard_LRS'
    }
    kind: 'StorageV2'
}
```
(zennさんbicepのコードハイライト実装もお願いします・・・！)

## .bicepparamファイルを作る
ここからが本題です。
param.bicepparamというファイルを作成します。（拡張子があってれば名前はなんでもOK）
.bicepparamファイルの先頭に、このパラメーターファイルがどのBicepファイルに対応しているかをusingで記述します。
![](/images/bicep-bicepparam-file/1.png)

ではパラメーターを記述していきます。具体的にはparamキーワードを使ってパラメーターを定義します。
![](/images/bicep-bicepparam-file/2.gif)
特徴として前述した
- 簡潔な記述方式
- テンプレートファイルに対するコード補完

が分かると思います。特に後者はusingによって使用するテンプレート(Bicep)ファイルを指定することにより、パラメーターファイルとテンプレートファイルを紐づけし、パラメータ名などを補完します。

ちなみに同じ内容のJSON形式と.bicepparam形式のパラメーターファイルを比較するとこんな感じです。文量が大幅に減り、可読性も向上しているのが分かると思います。
![](/images/bicep-bicepparam-file/compare.png)


## デプロイしてみる。
az cliからのデプロイは今までとほぼ変わりません。parametersの引数に.bicepparamファイルを指定するだけです。
`az deployment group create -g BicepRG --template-file exercise1.bicep --parameters param.bicepparam`

しばらくして、無事に設定した値とリージョンでストレージがデプロイされました。
![](/images/bicep-bicepparam-file/3.png)

# 関連する便利機能
ここからは.bicepparamファイルのサポートによって追加されたいくつかの関連便利機能を紹介します。
## .bicepparamファイルの自動生成
.bicepparamファイルをコマンドで自動生成することができる機能です。JSON形式のパラメーターファイルも生成することができましたが、それの.bicepparam版です。

`az bicep generate-params --file main.bicep --output-format bicepparam`
コマンドで生成することができます。 **が、記事執筆時点では「unrecognized arguments: --output-format bicepparam」というエラーが発生します。** バグの可能性もあるので引き続き確認して更新していきます。


JSON版については別途記事を執筆しています。
https://zenn.dev/microsoft/articles/bicep-generate-params

## .bicepparamファイル内で環境変数の使用が可能に
readEnvironmentVariableという関数が追加され、.bicepparamファイル内で環境変数を使用することができるようになりました。
`param storageNamePrefix = readEnvironmentVariable('some_key')`

たとえばVMのログインパスワードなど、パラメーターとは切り離して扱いたい値やCI/CD時の動的値の注入に利用できそうです。

## VS Codeの「Bicepファイルをデプロイ」アクションで.bicepparamファイルが選択可能に
Bicep拡張機能をインストールしている際にVS CodeのエクスプローラーでBicepファイルを右クリックすると「Deploy Bicep File」が表示され、選択すると対話型でデプロイのセットアップができます。この際に表示されるパラメーターファイルの選択肢に.bicepparamファイルが表示されるようになりました。
![](/images/bicep-bicepparam-file/5.png)

## VS Code拡張機能で「Buildparams」コマンドの追加
Bicep拡張機能をインストールして.bicepparamファイルを右クリックすると「Build Parameters File」が表示されます。
これをクリックすると.bicepparamファイルからJSON形式のパラメーターファイルを生成することができます。（ ~~なんのために使うんだ・・・？~~ ）
![](/images/bicep-bicepparam-file/4.png)

## .bicepparamファイルのデコンパイラー
JSON形式のパラメーターファイルを.bicepparamファイルに変換する機能です。（前述のBuildparamsコマンドの逆）
紐づくbicepファイルも指定してあげることでusingも自動で付与されるようです。
`az bicep decompile-params params.json --bicep-file main.bicep`

また、Bicep拡張機能をインストールしている際にjsonファイルを右クリックして表示される「Decompile into xxx」でも同様のことが可能です。
![](/images/bicep-bicepparam-file/6.png)

**が、こちらのデコンパイル機能も執筆時点ではうまく動きませんでした。今後の状況を確認していきます。**






# まとめ
正式リリース初日ということもあり、うまく動かない部分もありましたが、今後Bicepを使った開発では.bicepparamファイルを使うことが主流になると思います。今後の機能追加にも期待したいところです。

うまく動かない部分について同僚がGitHubにIssueを投稿してくれました。（私は+1しただけ）
https://github.com/Azure/bicep/issues/10978