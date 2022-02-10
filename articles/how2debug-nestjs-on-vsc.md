---
title: "VSCode で NestJS をデバッグする"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["NestJS", "VSCode", "Debug"]
published: false
---

# VSCode で NestJS をデバッグする

## TL;DR

1. `$ mkdir .vscode` でプロジェクトルートに `.vscode/` を作る
2. `$ touch .vscode/launch.json` でデバッグするために launch ファイルを作る
3. `package.json` に 任意の名前（ここでは `start:debug`）で `nest start --watch --debug` の npm scripts を追加
4. launch.json に以下を記述

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "pwa-node",
      "name": "Launch via NPM",
      "request": "launch",
      "runtimeArgs": [
          "run",
          "start:debug"
      ],
      "runtimeExecutable": "npm",
      "skipFiles": [
          "<node_internals>/**"
      ],
      "sourceMaps": true,
      "envFile": "${workspaceFolder}/.env",
      "cwd": "${workspaceRoot}",
      "console": "integratedTerminal",
      "protocol": "inspector"
    },
  ]
}
```

## 実際に使ってる様子

こんな感じで起動できる。

デバッグメニューに変数の中身やコールスタックが表示されていたらオッケー。

<!-- TODO: 実際にデバッグしてる画像を貼る -->


## ここがいい

- NestJS の `--watch` はよく知られてるけど `--debug` は文献がなかなかないが実は有益
    - というかいくら探しても公式のドキュメントが見つからない（どなたか見つけたら 🙏🏻）
- nodemon や ts-node-dev などは入れずに `--watch` で代用できるのは良い。
    - nodemon.json などを書きたくない。

## Context

デバッグは原始的な console デバッグもいいけど、規模が増えてきたり処理の流れを追いたい際に重用するよね。

個人的には、名だたる OSS なども `.idea` や `.vscode/` は git 管理対象にしているとこも多くみるのでプロジェクトで入れちゃっていいと思う。

### settings.json を git 管理対象にするのは注意

ただ、 `.vscode/settings.json` だけは少し注意が必要。

例えば、僕は Peacook というエディタの枠を色付けするプラグインを使ってる。

これは、僕が普段から複数の VSCode プロジェクトを立ち上げているので、切り替えた際にどのプロジェクトかをプロジェクト名で識別するよりかは直感的に色で識別するためである。

まあ、こんな理由から使っているのだけど

Peacook はワークスペース（プロジェクト）ごとに `settings.json` に任意の色コードを保存する。

そのため、仮に他にも Peacook を使っているユーザがいると、色コードが競合してしまう。

## FYI

[TypeScript +NestJSをプロジェクトで導入したら素晴らしかった件](https://zenn.dev/naonao70/articles/a91d8835f1832b)

[Cannot debug nest js application in VSCode](https://stackoverflow.com/questions/66535341/cannot-debug-nest-js-application-in-vscode)

[Debug Node.js Apps using Visual Studio Code](https://code.visualstudio.com/docs/nodejs/nodejs-debugging)
