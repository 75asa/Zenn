---
title: "Notion で日報を書くためにしたこと"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Notion", "Slack", "TypeScript", "Node.js"]
published: false
---

# 日報書いてますか？

弊社では日報を毎日書く文化があり、業務のあれこれや日々感じたことなどを各々が綴っています。
2 − 3 ヶ月ほど前までは社内 Wiki として Kibela があったので各自そこで日報を書いていました。
というのは建前で、僕個人としては 1 年半ほど日報を書かなかったのです。
というのも案件が忙しくなったり、めんどくさくなって一度サポってから全然手につかなくなりました。
また分報ではかなりアクティブにつぶやいているのでそれでいいかとかまけてしまっていました。

そんな弊社で社内 Wiki を Kibela から Notion に切り替えようという動きがありました。
詳しくはこちらの記事をご参照ください。

https://zenn.dev/75asa/articles/kibela-to-notion

そこで、これまで Kibela で書いてた日報も Notion に移行しようという運びとなりました。
今回の記事は Notion でどのように日報を書き、また運用していく過程で遭遇したあれこれや開発日記を書いていこうと思います。

# Kibela での日報体験

Kibela での体験は以下の通りです。
豊富な機能が充実していました。

- Webhook で気軽に Slack に通知ができる
- 投稿ボタンを押せば記事が公開されること
- フォルダ・グループで記事をカテゴライズできる
- 下書き状態ができる
- コメントができる

## 必要な要件

そこで比較して必要な要件を整理しました。

- 投稿ボタンを押して記事が公開されること
- Slack 通知
- 部署や案件などでフィルタできること

## 仕様

### main となる日報データベースで作成する
