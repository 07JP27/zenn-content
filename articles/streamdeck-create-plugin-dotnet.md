---
title: "Stream DeckのプラグインをC#で開発する"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["streamdeck", "dotnet", "csharp"]
published: true
---
:::message
この記事に記載されている内容はmacOSを使用していますが、Windowsでも可能です。
:::

# はじめに
[Elgato Stream Deck](https://www.elgato.com/ja/stream-deck)はライブストリーミング配信等での使用を主目的としたマクロキーパッドです。
Stream Deckでは様々な機能をプラグインとしてインストールして使用することができます。
公式でサポートされているプラグイン開発のための言語はJavascript, C++, Objective-Cですが、サードパーティのSDKを使用することでC#での開発も可能です。

## 前提条件
- [Dotnet Core SDK](https://get.dot.net/)(Version 2.2.100以上)
- [Stream Deck Software](https://www.elgato.com/gaming/downloads)


# 準備
C#用のSDKは「FritzAndFriends/StreamDeckToolkit」を使用します。
https://github.com/FritzAndFriends/StreamDeckToolkit
## プロジェクトテンプレートのインストール
```
dotnet new -i StreamDeckPluginTemplate
```

## プロジェクトの作成
```
dotnet new streamdeck-plugin --plugin-name MyPluginName
```

# 開発
テンプレートから作成したプロジェクトをVisual Studioで開きます。
![](https://storage.googleapis.com/zenn-user-upload/2093c2b187052e5adb79c3eb.png)
## ディレクトリ構成
- images
  プラグイン内で使用する画像の格納ディレクトリ
- models
  Stream Deckと情報をやり取りするモデルクラスのディレクトリ
- property_inspector
  プラグインの設定UIを構成するHTML,CSS,JSのディレクトリ
- manifest.json
  プラグインのメタ情報を記述するファイル
- MyPluginAction.cs
  プラグインの挙動が実際に記述してあるファイル

一部省略

## ロジックの開発
### ベースクラス
プラグインを作成する際には、Stream Deckボタンが実行するアクションを作成する必要があります。アクションを作成する際、継承できるベースクラスは2つあります。
#### BaseStreamDeckAction 
このクラスには、Stream Deckとの通信に必要なすべての統合機能が、「素」のレベルで含まれています。このクラスを継承することで、アクションがソフトウェアからデータを送受信する方法を最大限にコントロールすることができます。
```cs
public class MyPluginAction : BaseStreamDeckAction
{ 
  ....
}
```

#### BaseStreamDeckActionWithSettingsModel<T>
このクラスはBaseStreamDeckActionを継承し、モデルプロパティの生成を自動化します。
プロパティ **SettingsModel** は自動的にクラスTのインスタンスを生成しますので、クラスTを定義する際にはデフォルト値を割り当てることが推奨です。
また、Property Inspectorを使用してデータをやり取りする際には、JavaScriptのsettingsModelで定義されたプロパティとTで定義されたプロパティが一致している必要があります。
```cs
public class MyPluginAction : BaseStreamDeckActionWithSettingsModel<Models.CounterSettingsModel>
{ 
  ....
}
```

### ActionUuid
プロジェクトには、上記のいずれかのクラスを継承したアクションをいくつでも含めることができます。
アクションが起動時に自動的に登録されるためには、`[ActionUuid(Uuid="com.yourcompany.plugin.action.DefaultPluginAction")]`属性（UUIDは固有のものに変更）を持つ必要があります。
```cs
[ActionUuid(Uuid="com.yourcompany.plugin.action.DefaultPluginAction")]
public class MyPluginAction : BaseStreamDeckActionWithSettingsModel<Models.CounterSettingsModel>
{ 
  ....
}
```

Actionは`manifest.json`ファイルにActionUuid属性と一致するUuidを使って手動で登録する必要があります。
![](https://storage.googleapis.com/zenn-user-upload/8ef5d6ba3db3ea4f778d56c6.png)


## UIとロジックの接続
`property_inspector/property_inspector.html`で設定モデルクラスとマッピングするプロパティ名を指定します。
![](https://storage.googleapis.com/zenn-user-upload/cf80256c0c639090ad22d1ef.png)
より詳細なサンプルは[公式GitHub](https://github.com/elgatosf/streamdeck-pisamples)や[公式サイト](https://developer.elgato.com/documentation/stream-deck/sdk/property-inspector/)に掲載されています。
設定項目追加などでSettingModelが変わる場合は`property_inspector/js/property_inspector.js`の内容を変更してマッピングされるようにします。（これが地味にめんどくさかった）

# デバッグ実行
**Visual Studioで一度ビルドした後に**、テンプレートを使用してプロジェクトを作成した際に生成される`RegisterPluginAndStartStreamDeck.sh`を実行します。(Windowsの場合は.ps1の方)
```sh
sh RegisterPluginAndStartStreamDeck.sh
```
使用するSDKのバージョンによってはビルド成果物のファイルパスが異なるため、そのまま実行するとエラーが発生します。その場合はshファイルを開いて以下のようにパスの部分を実際のパスに変更します。
```sh
cp -R "$projectDir/bin/Debug/netcoreapp2.2/osx-x64/." $pluginName.sdPlugin
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
cp -R "$projectDir/bin/Debug/netcoreapp3.1/osx-x64/." $pluginName.sdPlugin
```
Stream Deckの設定アプリを開くと`manifest.json`で指定したメタ情報でプラグインが登録されているのでドラッグ&ドロップしてキーに割り当てます。
![](https://storage.googleapis.com/zenn-user-upload/d5b89397997ece465be11472.png)

# リリースビルド
1. Stream Deckの公式開発サイトからDistributionToolを[ダウンロード](https://developer.elgato.com/documentation/stream-deck/sdk/exporting-your-plugin/#download)します。
1. プラグインのインストールディレクトリにでDistributionToolを使用して.sdPluginディレクトリをパッケージします。
```sh
cd {HOME}/Library/Application Support/com.elgato.StreamDeck/Plugins
DistributionTool -b -i com.yourcompany.plugin.action.sdPlugin -o ~/Desktop/
```
3. 指定した場所（この例ではDesktop）に`.streamDeckPlugin`ファイルがエクスポートされます。

# 配布
## ストア上で
将来的に開発者ポータルでアップロードできるようになる予定のようですが、この記事の執筆時点では窓口にコンタクトする必要があります。
詳細は[公式サイト](https://developer.elgato.com/documentation/stream-deck/sdk/distribution-on-the-store/)をご確認ください。

## 独自の方法で
エクスポートした`.streamDeckPlugin`を独自サイトにアップロードするなどして配布します。
ユーザーは`.streamDeckPlugin`をダウンロードして開くとStream Deckアプリケーションが自動的にプラグインをインストールします。


# 参考
[How to Create an Elgato Stream Deck Plugin Using C#](https://www.youtube.com/watch?v=yxtxwlnUCws)(Youtube) 