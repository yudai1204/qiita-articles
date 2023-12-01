---
title: VSCodeの拡張機能でステータスバーにアイコンボタンを表示する
tags:
  - ""
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

:::note info
この記事は**デジクリ Advent Calender 2023** の 6 日目の記事です。
デジクリは芝浦工業大学の創作サークルです。
デジクリについて、詳しくは[こちらのサイト](https://digicre.net/)をご覧ください。
5 日目の記事は (19 th) さか さんの「**限界芝浦生に贈る、はじめての快活泊ガイド**」です。
:::

こんにちは、[うだい](https://twitter.com/yudai1204)です。
皆さんは VSCode を使っていますか？
VSCode は、それだけでも非常に使いやすくポピュラーなエディタの 1 つですが、**拡張機能**をインストールすることで、様々な機能を追加することができます。
今回は、VSCode の拡張機能を実際に開発し、その中でもステータスバーにアイコンボタンを表示する方法を紹介します。

## VSCode の拡張機能開発について

VSCode の拡張機能は、JavaScript や TypeScript で比較的簡単に開発することができます。
特に、yo コマンドを使うことで、拡張機能の雛形を作成することができます。

```shell:terminal
$ npm install -g yo generator-code
$ yo code
```

> `yo`コマンドを実行すると変なおじさんが出てくる ↓  
> <img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449776/710bf12f-c0a0-d188-58cf-8adeb3ddcb31.png">

対話型でいくつか質問されるので、適宜答えていきます。
今回は、拡張機能の名前は`my-extension`で作成します。
また言語は `JavaScript` か `TypeScript` から選択できますが、今回は `TypeScript` を選択します。

生成が完了したら、ライブラリをインストールします。

```shell:terminal
$ npm install
or
$ yarn install
```

これで、拡張機能の開発環境が整いました。
では、このディレクトリを VSCode で開きましょう。

うまく生成できると、`src`ディレクトリには、`extensions.ts`という拡張機能のコードが入っています。

```typescript:src/extensions.ts
// The module 'vscode' contains the VS Code extensibility API
// Import the module and reference it with the alias vscode in your code below
import * as vscode from 'vscode';

// this method is called when your extension is activated
// your extension is activated the very first time the command is executed
export function activate(context: vscode.ExtensionContext) {

    // Use the console to output diagnostic information (console.log) and errors (console.error)
    // This line of code will only be executed once when your extension is activated
    console.log('Congratulations, your extension "my-extension" is now active!');

    // The command has been defined in the package.json file
    // Now provide the implementation of the command with registerCommand
    // The commandId parameter must match the command field in package.json
    let disposable = vscode.commands.registerCommand('my-extension.helloWorld', () => {
        // The code you place here will be executed every time your command is executed
        // Display a message box to the user
        vscode.window.showInformationMessage('Hello World from my-extension!');
    });

    context.subscriptions.push(disposable);
}

// this method is called when your extension is deactivated
export function deactivate() {}

```

この最初からあるコードは、拡張機能の実行コマンドが走った時に`Hello World`を表示するシンプルなテンプレートです。

これを実行するには、VSCode 上で`F5`キーを押すか、`デバッグ`メニューから`デバッグの開始`を選択します。
VSCode 内に拡張機能のビルド機能があるので、たったこれだけでこの新しく作った拡張機能を実行することができます！！

## ステータスバーにボタンを表示する

VSCode の拡張機能開発では様々な API が提供されておりますが、今回はその中でもステータスバーにボタンを表示することについてフォーカスしていきます。

VSCode の**ステータスバー**(Status Bar)は、画面下部に表示されているバーのことです。

![status-bar.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449776/3f1a1844-c7b0-829b-eed8-0dc3a6967cf4.png)

> 画像引用 - [Status Bar - Visual Studio Code](https://code.visualstudio.com/api/ux-guidelines/status-bar)

ステータスバーには拡張機能から任意の情報を表示することができ、またボタンとしてクリックイベントを設定することもできます。

`src/extensions.ts`を開き、以下のコードに置換してみましょう。

```typescript:src/extensions.ts
import * as vscode from "vscode";
import { window, StatusBarAlignment } from "vscode";

export function activate(context: vscode.ExtensionContext) {
  const myButton = window.createStatusBarItem(StatusBarAlignment.Left, 1000);
  myButton.text = "Click Here!";
  myButton.command = "my-extension.helloWorld";
  myButton.show();

  let disposable = vscode.commands.registerCommand(
    "my-extension.helloWorld",
    () => {
      vscode.window.showInformationMessage("Hello World from my-extension!");
    }
  );

  context.subscriptions.push(disposable);
}
export function deactivate() {}

```

また、`package.json`を開き、VSCode の起動と同時に拡張機能自体が読み込まれるようにしましょう。

```diff_javascript:package.json
-  "activationEvents": [],
+  "activationEvents": [ "*" ],
```

このコードを実行すると、ステータスバーに`Click Here!`というボタンが表示されます。
そして、このボタンをクリックすると、`Hello World from my-extension!`というメッセージが表示されます。
とてもシンプルですね。

## ステータスバーに任意のアイコンを表示する

前置きが長くなりましたが、ここから本題に入ります。
本題にある、「**ステータスバーに任意のアイコンを表示する**」方法について紹介します。

私は以下の拡張機能を制作して公開しています。

> Open in Fork Button

https://marketplace.visualstudio.com/items?itemName=yudai1204.open-in-fork-button

この拡張機能は、VSCode で開いている Git のレポジトリを私の愛用する Git クライアントである[**Fork**](https://git-fork.com/)で開くことができるようにするものです。

> Git クライアント - Fork

https://git-fork.com/

導入すると、以下の画像のように VSCode のステータスバーにボタンが表示され、ここをクリックすることで該当のレポジトリを`Fork` で開くことができます。
Windows, Mac と WSL2 の Ubuntu で動作します。

> <img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449776/cb41a15e-3ee9-ec84-cb7e-4ee97a68461d.png">

このスクリーンショットを見るとわかるように、このボタンには`Fork`のアイコンが表示されています。
しかし、**VSCode の標準機能ではステータスバーに任意の画像を直接表示する方法は用意されていません。**

そこで、この拡張機能では**オリジナルフォントを作成し、文字としてアイコンを表示する**という方法を取っています。

### オリジナルフォントを作成する

#### 1. アイコン画像を用意する

まずはアイコンとなる svg 画像を用意します。
png や jpg しかない場合は、[Vectorizer](https://www.vectorizer.io/)などを使って svg に変換してください。

今回は、以下のようなアイコンを用意しました。

<img width="100" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449776/c55b64f9-ef8c-9530-bbf9-d3f086255a6e.png">

文字として扱うため、白黒の画像にしておきます。
また、文字色は黒、背景色は透明もしくは黒として設定しておくとフォントへの変換時に正しく変換されやすいです。

#### 2. アイコン画像をフォントに変換する

次に、svg 画像をフォントに変換します。
変換には、[IcoMoon](https://icomoon.io/)を使います。

https://icomoon.io/

IcoMoon にアクセスし、`IcoMoon App`を開きます。
`Import Icons`をクリックし、先ほど用意した svg 画像を選択します。
うまく読み込まれると、以下のように画像が表示されます。

<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449776/33353ac0-e02d-85b3-656b-b61c9f128660.png">

画面上部のサーチボッスの横にある鉛筆アイコンをクリックし、`Edit`を選択してから、先ほどアップロードして表示されているアイコンをクリックします。
以下のようなモーダルが表示されるので、`Tags`および`Names`には任意のフォント名を入力します。日本語は使えないので、英語で入力しましょう。
そして、赤矢印で示されるように`Fit to Canvas`をクリックして、余白を削除します。
これをしないとアイコンが小さく表示されてしまいますので、忘れずに行うようにしましょう(一敗)。

<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449776/566f5820-2fb0-2c8f-b251-513db1a506cc.png">

モーダルの外をクリックすると自動的に保存されます。
保存されたら、画面右下の`Generate Font`をクリックします。

以下のような画面が表示されるので、作成したアイコンの文字コード`e900`になっていることを確認します。
`e900`以外を使用しても特には問題ないのですが、[Unicode における`U+E900`は私的領域に指定されている](https://0g0.org/unicode/E900/)ためこちらを使用することにします。

![スクリーンショット 2023-12-01 14.09.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449776/d9ef18b8-7632-4252-3317-332eea2d543c.png)

問題なさそうであれば画面右下の`Download`をクリックし、zip ファイルをダウンロードします。

#### 3. フォントを拡張機能に読み込ませる

ダウンロードした zip ファイルを解凍し、`fonts`ディレクトリにある`icomoon.woff`を拡張機能のルートディレクトリ直下に`assets`ディレクトリを作り、そこにコピーします。
そして、`package.json`を編集してフォントを使用できるようにします。

`package.json`の`contributes`の中に`icons`という項目を追加し、`digicre-logo`という名前で`fontPath`にフォントのパス、`fontCharacter`に文字コードを指定します。
ここで、名前はハイフンを 1 つ以上含む必要があります。
また、`fontCharacter`は先ほど icomoon で設定した文字コードを指定します。

```diff_javascript:package.json
...
"main": "./out/extension.js",
"contributes": {
+  "icons": {
+    "digicre-logo": {
+      "description": "Digicre Icon",
+      "default": {
+        "fontPath": "assets/icomoon.woff",
+        "fontCharacter": "\\E900"
+      }
+    }
+  },
  "commands": [
    {
      "command": "my-extension.helloWorld",
      "title": "Hello World"
    }
  ]
},
"scripts": {
  "vscode:prepublish": "npm run compile",
...
```

次に、`src/extensions.ts`を編集します。
`activate`関数の中の以下の部分を変更します。

```diff_typescript:src/extensions.ts
...
export function activate(context: vscode.ExtensionContext) {
  const myButton = window.createStatusBarItem(StatusBarAlignment.Left, 1000);
-  myButton.text = "Click Here!";
+  myButton.text = "$(digicre-logo) Click Here!";
  myButton.command = "my-extension.helloWorld";
  myButton.show();
...
```

`myButton.text`の値に`$(digicre-logo)`を追加することで、先ほど`package.json`で設定した`digicre-logo`という名前のアイコンを表示することができます。

これで、ステータスバーにアイコンを表示することができました！

<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449776/fd4620ef-8f7b-1326-75d1-b9a9391287c9.png">

## 終わりに

ここまで読んでくださりありがとうございました。

VSCode の拡張機能開発は、JavaScript や TypeScript で比較的簡単に開発することができます。
その中でステータスバーボタンは特によく使われる機能です。
その分、ステータスバーには多くのボタンが表示され、アイコンを使って 1 つ 1 つを短く表示したいことがあります。
今回紹介した方法をぜひ活用して、オリジナルの拡張機能を制作してみてください！

使用した拡張機能のソースコードは以下のリポジトリにあります。

https://github.com/yudai1204/digicre-vscode-test-extension

明日の記事は本日に引き続き私、うだい の「猫でもわかる OpenAI API」です。
お楽しみに！
