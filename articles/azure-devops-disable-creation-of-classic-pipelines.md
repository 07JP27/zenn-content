---
title: "Azure DevOpsでクラシックパイプラインの作成を禁止する"
emoji: "🚫"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['azure', 'azuredevops']
published: true
publication_name: "microsoft"
---


# TL;DR
- Azure DevOpsでクラシックパイプラインの作成を禁止する設定がリリースされました。
- **2023年3月以降に作成される組織は既定で禁止設定になる予定です。**

本記事は、こちらのブログの紹介です。
https://devblogs.microsoft.com/devops/disable-creation-of-classic-pipelines/

# クラシックパイプライン vs YAMLパイプライン
Azure DevOpsのパイプラインは、クラシックパイプラインとYAMLパイプラインの2種類があります。現在の主流は名前にも現れている通り、YAMLパイプラインです。
YAMLパイプラインはクラシックパイプラインと比較して以下のようなメリットがあります。
- **コードレビューが容易**
    YAMLパイプラインは文字通りテキストベースのYAMLで記述され、リポジトリによってバージョン管理やコードレビューができます。
- **リソースアクセス管理を提供**
    パイプラインが依存する、環境、サービス接続、エージェントプール、変数グループ、セキュリティで保護されたファイルなどのリソースを使用できるかどうかを制御できます。
- **[ランタイムパラメーター](https://learn.microsoft.com/ja-jp/azure/devops/pipelines/process/runtime-parameters?view=azure-devops&tabs=script)をサポート**
    ランタイム パラメーターによって、パイプラインに渡すことができる値をより詳細に制御できます。

# クラシックパイプラインの作成を禁止する
セキュリティを重視する場合は、YAMLパイプラインのみを使用することを選択する事が多いです。今まで開発者に対してYAMLパイプラインの作成を強制できませんでしたが、今回この設定が可能になりました。

この設定は以下のスコープで設定できます。
- Organization
- Project

設定の継承が有効であり、以下のような動作をします。
- **Organizationレベルでオンの場合**
    Organizationに属する全てのProjectでクラシックパイプラインの作成が禁止されます。
- **Organizationレベルでオフの場合**
    個々のProjectの設定内で設定できます。

## 方法
Organization SettingsまたはProject Settings画面の `Pipelines` → `Settings` から、`Disable creation of classic build and classic release pipelines` をオンにします。

![](/images/azure-devops-disable-creation-of-classic-pipelines/1.png)



# 既存のクラシックパイプラインはどうなる？
新規作成の禁止がオンになっている場合でも引き続き編集・実行が可能です。ただし新規作成することはできません。

この機会にクラシックパイプラインをYAMLパイプラインに移行する場合は、YAMLエクスポート機能を使用して移行することが可能です。
https://learn.microsoft.com/ja-jp/azure/devops/pipelines/migrate/from-classic-pipelines?view=azure-devops

# 2023年3月に既定で禁止設定になります
Microsftが既存の組織やプロジェクトに対して強制/自動的にこの機能をオンにする予定はありませんが、**2023年3月以降に作成される組織**については、この機能を**デフォルトでオン**にする予定です。
つまり、それ以降に作成される組織は、デフォルトでYAMLのみになります。何らかの理由があってクラシックパイプラインを使用する場合はこの設定項目を`オフ`にします。