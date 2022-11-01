---
title: "`firebase init hosting:github` から user/repo を入力しても CLI がうんともすんとも言わないとき"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["firebase", "github"]
published: true
---

# PreRequisite

- firebase hosting を使っている
- 個人リポジトリではなく org で使用している

# TL;DR

- 原因分析のためにコマンドの末尾に `--debug` をつけましょう
  - e.g. `$ firebase init hosting:github --debug`
- 404 error が返却されている場合 org に Firebase CLI の権限が Grant されていないので Grant しましょう
- https://github.com/settings/installations の `Authorized OAuth Apps` から Firebase CLI があるか確認しましょう
- 以下の画像のように x と表示されている場合 Grant を選択し管理者に Approve してもらうか管理者自身が Grant すれば OK です。
  ![GitHub Organization Access](https://i.gyazo.com/44a71c7a9c4644767aee0095de06a647.png)

# Context

firebase hosting で個人リポジトリでは行なっていましたが org では初めてでその際に調査したことのメモです。
公式の Actions は以下です。

https://github.com/marketplace/actions/deploy-to-firebase-hosting

# Note

- Org の Integrations ではなく個人アカウントの Integration から Grant する

# FYI

- https://github.com/firebase/firebase-tools/issues/2763
- https://github.com/firebase/firebase-tools/issues/3143
- https://qiita.com/ikedaosushi/items/eddd120aa108b1599a79
