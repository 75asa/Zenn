---
title: "`firebase init hosting:github` から user/repo を入力しても CLI がうんともすんとも言わないとき"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# PreRequisite
- firebase hosting を使っている
- 個人リポジトリではなく org で使用している

# Context
firebase hosting 楽でいいですよね。
GitHub Action での CI/CD も楽にできます。
ただ、 org で利用する場合はちょっと気をつけないといけません。
Official Document にある通り、 `firebase init hosting:github` で設定を行う必要があります。

# 1st Try
`$ firebase init hosting:github --debug` で `--debug` をつけましょう。

# 2nd Try
https://github.com/settings/installations の `Authorized OAuth Apps` から Firebase CLI があるか確認しましょう。

# 3rd Try
Firebase CLI から firebase deploy を組み込みたい Org にアクセス許可があるかを確認しましょう。
ない場合は、 Grant しましょう。
