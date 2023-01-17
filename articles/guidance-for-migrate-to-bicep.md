---
title: "Azureの既存リソースをBicep（IaC）化するガイダンス"
emoji: "🦾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'iac', 'bicep']
published: true
publication_name: "microsoft"
---

PoCやテストフェーズでAzrure Poratalを使って試行錯誤の末に完成させたリソース構成を「IaC化して管理したい！」というときはどうしていますか？

もちろんAzure Portalと睨めっこして、手書きでリソースを定義していくこともできます。しかし、すでに既存のリソースがあるのであればそれを最大限活用しましょう！
あまり知られていないかもしれませんが、Azureのドキュメントには「Bicepへ移行する」というガイダンスがあります。
https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/migrate

このガイダンスは大きく分けて２つのシナリオで活用できる内容になっています。
1. Azure Portal上で作成したリソースをBicep化する
1. すでに存在するARMテンプレートをBicep化して綺麗にする

本質的に上記の２つは **「ARMテンプレートをBicep化して可読性や管理のしやすさを向上させる」** という点で同じです。
今回はこのガイダンスを確認しながら、Azure Portalで作成したリソースをBicep化してみたいと思います。

# 想定読者
- Bicepとはなにか、Bicepの記載方法について基本的な知見がある方
    Bicepについては[こちらのドキュメント](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/overview?tabs=bicep)がわかりやすいです。

# 技術的前提
前提としてARMテンプレートとBicepの関係性を理解しておくとガイダンスが読みやすいです。
ARMテンプレートとBicepはどちらもAzureのリソースを定義する言語ですが、それぞれ独立したものではなく相互に変換ができ、互換性があります。
ARMテンプレートを使おうがBicepを使おうが、Azure内部でリソースはARMテンプレート形式(JSON)で管理されています。つまり、Bicepは人間が読み書き/管理しやすくするために作られた言語ともいえます。
以上のような背景からBicep→ARMテンプレートへの変換はBuildと呼ばれる順方向です。
逆に、ARMテンプレート→Bicepへの変換はDecompileと呼ばれる逆方向になります。
![](/images/guidance-for-migrate-to-bicep/bicep-arm.png)

:::message alert
ただし互換性があると言っても、逆方向のDecompileではベストエフォートでの変換となり完全性が保証されていません。Decompileした際は必ず人間が確認する必要があります。
![](/images/guidance-for-migrate-to-bicep/decompile-notice.png)
[出典](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/decompile?tabs=azure-cli#decompile-from-json-to-bicep)
:::

# ガイダンスの確認
では実際にガイダンスの内容を確認てみましょう。
https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/migrate

このガイダンスのモチベーションは**Azure環境に破壊的変更を加えるリスクを最小限に抑えながらテストとデプロイを行う**ことです。
具体的にはそれぞれのフェーズ内でより細かいタスクがありますが、大きく分けると５フェーズで移行を行います。
1. **変換**
    Azure PortalからARMテンプレートをエクスポートするか既存のARMテンプレートを用意してDecompileします。
1. **移行**
    前述の通りDecompileは完全性が保証されません。ここではデプロイ可能なBicepファイルの最初のドラフトを作成し、移行対象のAzureリソースが確実に定義されていることを確認します。
1. **リファクタリング**
    このステップではBicepコードの品質を向上させるのが目標です。モジュール化やコメントの追加(ARMテンプレートはJSONなのでコメントが書けないがBicepは可能)などを行い、より読みやすく、管理しやすいBicepファイルに仕上げます。
1. **テスト**
    移行されたテンプレートの整合性を検証し、デプロイテストを実行することを目標とします。
1. **デプロイ**
    最終的に運用環境にBicepファイルをデプロイします。

![](/images/guidance-for-migrate-to-bicep/migrate-bicep-overview.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/migrate)

# 今回のシナリオ
今回は以下のようなリソース構成をBicep化します。
プライベートエンドポイントを有効化したシンプルなWeb Apps環境です。そこにアクセスするためのVNetやVMなど周辺リソースもあります。
![](/images/guidance-for-migrate-to-bicep/senario.png)

それでは実際にガイダンスに従ってBicep化していきます。
## フェーズ1:変換
### 1-1. Azureリソースのキャプチャ
すでにARMテンプレートがある場合はそれを用意します。
今回はAzure Portal上で作成した既存リソースからARMテンプレートをエクスポートすることで既存リソースのARMテンプレートを用意します。
と言ってもそれほど難しいことはなく、対象のリソースが含まれたリソースグループの「テンプレートのエクスポート」からテンプレートを生成して「ダウンロード」を行うだけです。リソースグループに含まれているリソースがARMテンプレートの形式で出力されます。
![](/images/guidance-for-migrate-to-bicep/export-template.png)

「エクスポートしたARMテンプレートがそのまま使えるじゃん！」と思う方もいらっしゃると思います。それはそれで間違いではありません。しかし、Azureからエクスポートしたままの状態だと非常に読みづらいテンプレートになっています。この状態のまま構成を少し改変したりする際は不整合が発生したり「変えるべきところを見落とした」などミスをする可能性が高いです。さらにモジュール化などもされていないため、エクスポートしたリソースと完全に同じ構成でそのままデプロイしない限りは、将来への投資だと考えてBicep化とリファクタリングをお勧めします。
ガイダンス内にも記載されていますが、**優れたBicepコードとは、"自己文書化" されたコードです。**

### 1-2.ARMテンプレートのDecompile
次にARMテンプレート(JSON)からBicepファイルへDecompileします。
```
az bicep decompile --file template.json
```

Azure Portalからエクスポートした場合、ディレクトリ構造はこのようになっていると思います。
![](/images/guidance-for-migrate-to-bicep/exported-structure.png)

`az bicep decompile`コマンドを実行すると同じディレクトリにというファイルが生成されます。
![](/images/guidance-for-migrate-to-bicep/decompiled-structure.png)

再三になりますが、Decompileはベストエフォートです。`az bicep decompile`コマンドを実行した際も注意が表示されます。
![](/images/guidance-for-migrate-to-bicep/decompile-warning.png)


## フェーズ2:移行
### 2-1.新しい空の Bicep ファイルを作成する。
DecompileしたBicepファイルをそのまま変更するのではなく、真っ新なBicepファイルを作成して移行を行います。
新しいファイルの場所はどこでもいいですが、今回は同じディレクトリにmain.bicepというファイル名で作成しました。

### 2-2.逆コンパイルされたテンプレートから各リソースをコピーする
各リソースを個別にコピーします。 このプロセスは、リソースごとに問題を解決します。
`template.bicep`から`main.bicep`へどんどんコピペしながらエラーなどの問題を解決していきます。この時にVSCodeのSplit Viewを使うと便利です。
![](/images/guidance-for-migrate-to-bicep/vscode-split-view.png)

### 2-3.不足しているリソースを特定して再作成する。
`main.bicep`をざっと眺めてみて不足してそうなリソースがないか確認し、不足があれば手書きで追加します。たとえば仮想マシン拡張機能はエクスポート用にサポートされていないので、必ず再作成する必要があります。
私の場合はVNetにサブネットの定義が入っているのに別途サブネットリソースが同じ構成で定義されていたりしたので、このような不足だけではなく過剰な部分もここで削除しておきます。
またVSCodeを使用して[Bicep拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep&azure-portal=true)をインストールしている場合はBicepファイルの視覚化ボタンを押すことで大まかなリソースの関連が見れます。
![](/images/guidance-for-migrate-to-bicep/bicep-visualize.png)
![](/images/guidance-for-migrate-to-bicep/topology.png)

問題ないようであればビルドをしてみます。
```
az bicep build --file main.bicep
```
警告が出ている部分はリファクタの段階で修正するので、ここではエラーなくビルドが通るかどうかを確認します。ビルドに成功すると同じ名前で拡張子がjsonのARMテンプレートファイルが生成されます。
![](/images/guidance-for-migrate-to-bicep/build-bicep.png)

フェーズ2ではパラメーター名の変更などはせずに最低限ビルド(デプロイ)可能な状態にだけ改変するようにガイダンスに記載がありますが、ビルドを通すことすらもままならないくらいテンプレートがごちゃごちゃな場合は、この段階でリファクタを少しだけ始めてもいいかもしれないと思いました。
特にリソース名を変更したり、何のリソースかコメントを残しておかないとリソースの整理ができませんでした。(リソースはsubnetなのにリソース名はvnetになっていたり・・・)

## フェーズ3:リファクタリング
### 3-1.リソースAPIバージョンのレビュー
各リソースにはAPIバージョンが決まっており、BicepではどのバージョンのAPIを使用するかを明示します。
Azure Portalからエクスポートした状態だとAPIが最新になってなかったり、追加で使いたいプロパティを含んでいないAPIバージョンになっていることがあるので、APIのバージョンを再度確認します。[VSCode用のBicep拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep&azure-portal=true)をインストールしている場合はそのリソースで使えるバージョン一覧がインテリセンスとして表示されます。
![](/images/guidance-for-migrate-to-bicep/arm-api-intelisence.png)

### 3-2.リンターの修正候補をレビュー
[VSCode用のBicep拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep&azure-portal=true)をインストールしている場合はリンターが自動的に実行され、コードのエラーと警告が表示されます。エラーの部分はフェーズ２で修正したと思うので、ここでは警告の確認と修正を行います。
警告はBicepのベストプラクティスに沿っていない場合に表示され、たとえばリソースのロケーションを固定値で記載していると「ロケーションがユーザーが選択できるべきである」と警告されます。Azure Portalからエクスポートしたテンプレートはエクスポート対象のリソースがが配置されていたリージョンが固定値で記載されているので、これをパラメーター化します。
![](/images/guidance-for-migrate-to-bicep/lint-warn.png)
![](/images/guidance-for-migrate-to-bicep/refact-location.png)

### 3-3.パラメーター、変数、およびシンボリック名を変更
デコンパイラによって生成されるパラメーター、変数、およびシンボリック名の名前を確認し、必要に応じて調整を行います。パラメーターの初期値はエクスポート対象のリソースの値が入っているのでここから何に使われているパラメーターかを予測して名前の変更を行なっていきます。このタイミングでVNetのアドレス空間など、パラメーターを新規追加することもできます。
![](/images/guidance-for-migrate-to-bicep/exported-params.png)

### 3-4.式の簡略化
例えばconcat関数は文字列補完という記法で簡略化できます。このように簡略化できる式も[VSCode用のBicep拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep&azure-portal=true)のリンターによって修正候補が確認できます。
```
concat(prefix, 'And', uniqueString(resourceGroup().id)
↓
'${prefix}And${uniqueString(resourceGroup().id)}'
```

### 3-5.子および拡張リソースのレビュー
Bicepには、子リソースおよび拡張リソースを宣言する方法が複数あります。
VNetに含まれるSubnetはVNetリソースの定義内に含んで定義することもできますし、親をVNetとして独立したリソースとして定義することも可能です。
基本的にはどちらでも最終的な結果は変わりませんが、記述時点の制限として、子リソース名の作成に文字列の連結を使用しないようにするには、parentプロパティまたは入れ子になったリソースを使用する必要があります。

### 3-6.モジュール化
モジュール化は文字通りある部分のみをモジュールとして別Bicepファイルに分けて、それを大元のBicepファイルから参照する方法です。
Bicep モジュールを使用すると、コードの複雑さを軽減とともにBicepコードの再利用性を高めることができます。
例えばVNetだけをデプロイするジュール、WebAppsだけをデプロイするモジュールなど役割やリソースごとにモジュール化をしておくと将来的にリソースのカタログのような形で必要なリソースを組み合わせて様々な構成をデプロイすることができます。
https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/modules


### 3-7.コメントと説明の追加
パラメーターに@description() 属性を付与したり、適宜コメントを記載していきます。

### 3-8.Bicepのベストプラクティスに従う
Bicepにはベストプラクティスが用意されています。とはいえこの段階までくるとほとんどベストプラクティスに沿った形になっている場合もあると思います。
https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/best-practices

## フェーズ4:テスト
### 4-1.ARM テンプレートデプロイのWhat-If操作を実行する
実際にデプロイする前にWhat-If操作を行います。これは環境の現在の状態とテンプレートに定義されている状態を比較するコマンドです。
このツールでは、実際にはデプロイをせずにデプロイした場合に発生する変更の一覧が出力されるので、最終確認として実行すると良いでしょう。
増分および完全モードの両方のデプロイで、What-If を使用できますが、運用では増分モードを使用してテンプレートをデプロイする予定の場合でも、What-If操作を完全モードで実行することが推奨されています。
![](/images/guidance-for-migrate-to-bicep//what-if.png)
[引用元](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/templates/deploy-what-if?tabs=azure-powershell)

増分モードと完全モードについては過去に書いた以下の記事をご覧ください。
https://zenn.dev/07jp27/articles/azure-deploy-mode

### 4-2.テストデプロイを実行する
要するに、いきなり本番環境にデプロイするのはリスクがあるので、テスト環境を作ってそこにデプロイしてみましょう、というです。

## フェーズ5:デプロイ
### 5-1.ロールバック計画を準備する
仮にデプロイに失敗した場合にどのような対処を行うのかを検討します。

### 5-2.運用環境に対してWhat-If操作を実行する
最終的にBicepファイルを運用環境にデプロイする前に、テスト環境と同様に運用環境に対してもWhat-If操作を実行しましょう。運用パラメーターの値を使用して、既存のリソースにどのような変更があるのか結果を記録しておくことが推奨されています。

### 5-3.手動でデプロイする
最終的にはCI/CDパイプラインなどで自動デプロイすることもできますが、まずは手動でデプロイしてみてテンプレートが正常にデプロイできるかを確認することが推奨されています。

### 5-4.スモークテストを実行する
デプロイが完了したら、一連の "スモーク テスト" を実行し、アプリケーションまたはワークロードが正常に動作していることを確認します。
スモークテストの方法はワークロードごとによって異なります。例えばWebアプリの場合はブラウザからアクセスができるか、データが読み書きできるか、などです。

# MS Learn
実際に手を動かしてみたい方はMS Learnの「Bicep を使用するために Azure リソースと JSON ARM テンプレートを移行する」ハンズオンもお試しください！
https://learn.microsoft.com/ja-jp/training/modules/migrate-azure-resources-bicep/