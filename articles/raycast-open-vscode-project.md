---
title: "mac向け神ランチャーツールRaycastでVS Codeのプロジェクトを一発起動"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mac","raycast", "vscode"]
published: true
---

# Raycastとは
Raycastはmacで使用できるランチャーツールです。
ランチャーというと標準だとSpotlight、競合だとAlfredがありますが、RaycastはSpotlightよりも高機能かつAlfredだと有料版でしか使えない機能も無料だったり、と控えめに言っても最高なツールです。
![](/images/raycast-open-vscode-project/launch-apps.jpg)

ここではRaycast自体についてはあまり深く話しません。インストール方法や詳しい機能を知りたい方は以下の記事をご覧ください。
https://zenn.dev/fumi_sagawa/articles/2ff5fd9c03fbcd
https://zenn.dev/yum3/articles/i_raycast_or_alfred

# RaycastでVS Codeを開きたい！
「使わない日はない!」という方も少なくないのではないかというほどのシェアを誇るVS Code。
VS Codeで特定のプロジェクト(ディレクトリ)を開くときってどうしてますか？
- ターミナルでディレクトリに移動して`code .`?
- VS Codeのウインドウに開きたいディレクトリをドラック&ドロップ？

どちらもそれなりにスマートですが、複数ステップにわたるので何度も開く場合はちょっと面倒です。そこで「Raycastで特定プロジェクトを開きたい！」となります。(よね？)
RaycastはランチャーアプリなのでもちろんインストールされているVS Codeを起動できますが、プロジェクトを指定して開くことは標準ではできません。そんな時は拡張機能の出番です。
**本記事はRaycastと拡張機能を使用して特定のプロジェクトをVS Codeで開く設定について記載します。**
最初はちょっと設定をする必要がありますが１度設定してしまえば日々の作業で簡単に開けるようになります！

# 始める前に
## 前提条件
以下の準備が整った状態からスタートします。
- VS Codeがインストールされている
- Raycastがインストールされている

## 全体の流れ
全体のステップとしては大まかに4ステップです。
1. Raycastに拡張機能をインストール
1. VS Codeに「Project Manager」拡張機能をインストール
1. 「Project Manager」でプロジェクトを登録
1. 起動！

# やってみよう
## 1.Raycastに拡張機能をインストール
Raycastの`Preference`を開き`Extensions`→`+`→`Install From Store`をクリックします。
![](/images/raycast-open-vscode-project/0.png)
![](/images/raycast-open-vscode-project/1.png)

検索ボックスに「vscode」と入力します。`Visual Studio Code - Project Manager`を選択して`Command + Enter`キーでインストールします。
![](/images/raycast-open-vscode-project/2.png)

`Preference`を閉じてRaycastを起動します（デフォルトHotkeyは`Option + Space`キー）。
`vscode`と入力して`Visual Studio Code - Project Manager`が候補に表示されればインストールは完了です。
![](/images/raycast-open-vscode-project/3.png)

しかし、`Visual Studio Code - Project Manager`を選択してもプロジェクトは何も表示されないと思います。
これは、Raycastが「どのディレクトリ配下がプロジェクトなのか」を認識していないからです。
それを解決するために続いてVS Code側でプロジェクトの登録をします。


## 2.VS Codeに「Project Manager」拡張機能をインストール
VS Codeを起動します。
拡張機能(Extensions)メニューを開き、検索ボックスに`project manager`と入力し`Project Manager`拡張機能をインストールします。
![](/images/raycast-open-vscode-project/4.png)

## 3.「Project Manager」でプロジェクトを登録
プロジェクトとして開きたいディレクトリを`code .`や`ドラック&ドロップ`を使ってVS Codeで開きます。
今回はサンプルとして私がZennのコンテンツを管理しているディレクトリ(リポジトリ)の`zenn-content`を使用します。
`Command ＋ Shift ＋ P`キーでコマンドパレットを開きます。
`save project`と入力して`Project Manager:Save Project`を選択します。
![](/images/raycast-open-vscode-project/5.png)
プロジェクト名を入力します。デフォルトではディレクトリの名前になりますが、そのままでも変更してもどちらでもOKです。
![](/images/raycast-open-vscode-project/6.png)
プロジェクト名を入力したらEnterキーで登録し、`Project saved!`と表示されれば成功です。
![](/images/raycast-open-vscode-project/7.png)

Raycastでプロジェクトとして開きたいディレクトリに対して３の登録を繰り返します。

## 4.起動！
Raycastを起動します（デフォルトHotkeyは`Option + Space`キー）。
`Visual Studio Code - Project Manager`を選択します。
![](/images/raycast-open-vscode-project/8.png)
すると3で登録したプロジェクトが表示されるので開きたいプロジェクトを選択してEnterキーを押します。
![](/images/raycast-open-vscode-project/9.png)
選択したプロジェクトが開かれました！
![](/images/raycast-open-vscode-project/10.png)


# おまけ：プロジェクトの削除
登録したプロジェクトがいっぱいになってくるとユーザービリティが下がってしまうので、使用頻度が低くなったプロジェクトは登録を消しましょう。

VS Code内でコマンドパレットを開きます（`Command ＋ Shift ＋ P`キー）。
`edit project`と入力して`Project Manager:Edit Projects`を選択します。
![](/images/raycast-open-vscode-project/11.png)

`projects.json`ファイルが開きます。この事からも分かるように、このファイルを編集すれば登録してあるプロジェクトを編集できます。
削除したいプロジェクトのブロックを削除して`projects.json`ファイルを保存すれば削除完了です。
![](/images/raycast-open-vscode-project/12.png)



皆様、よいmacライフを！