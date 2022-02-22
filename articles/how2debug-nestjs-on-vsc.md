---
title: "VSCode で NestJS をデバッグする"
emoji: "🐛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["NestJS", "VSCode", "Debug"]
published: true
---

# VSCode で NestJS をデバッグする

## TL;DR

1. `$ mkdir .vscode` でプロジェクトルートに `.vscode/` を作る
2. `$ touch .vscode/launch.json` でデバッグするために launch ファイルを作る
3. `package.json` に 任意の名前（ここでは `start:watch`）で `nest start --watch` の npm scripts を追加
4. launch.json に以下を記述（npm 派は適宜変えてください）

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "pwa-node",
      "name": "Launch via yarn",
      "request": "launch",
      "runtimeArgs": [
          "run",
          "start:watch"
      ],
      "runtimeExecutable": "yarn",
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

こんな感じで起動できます。

デバッグメニューに変数の中身やコールスタックが表示されていたらオッケーです。

![Debugging on VSCode](https://i.gyazo.com/2bc5a9011e4ee2b90d4436b9792564ad.png)



## ここがいい

- nodemon や ts-node-dev などは入れずに `--watch` で代用できるのは良い
- nodemon.json などを書きたくない、楽したい

## Context

デバッグはプリミティブな console デバッグもいいけど、規模が大きくなったりコールスタック等から処理の流れを追いたい際に重用します。

## 余談

### `.vscode/` や `./idea` を Git 管理すべきか

個人的には、名だたる OSS なども `.idea` や `.vscode/` は Git 管理対象にしているとこも多くみるのでプロジェクトで入れちゃっていいと思います。


## Note
### settings.json を Git 管理対象にするのは注意

ただ、 `.vscode/settings.json` だけは少し注意が必要です。

例えば、僕は Peacook というエディタの枠を色付けするプラグインを使っています。

https://marketplace.visualstudio.com/items?itemName=johnpapa.vscode-peacock

これは、僕が普段から複数の VSCode プロジェクトを立ち上げているので、切り替えた際にどのプロジェクトかをプロジェクト名で識別するよりかは直感的に色で識別するためです。

しかし、 Peacook はワークスペース（プロジェクト）ごとに `settings.json` に任意の色コードを保存します。

そのため、仮に他に Peacook を使っているユーザがいると、色コードが競合してしまいます。

## FYI

https://zenn.dev/naonao70/articles/a91d8835f1832b

https://stackoverflow.com/questions/66535341/cannot-debug-nest-js-application-in-vscode

https://code.visualstudio.com/docs/nodejs/nodejs-debugging
