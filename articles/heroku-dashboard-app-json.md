---
title: "Heroku Dashboard から app.json を即座に出力する"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["heroku"]
published: false
---

# TL;DR

1. `https://dashboard.heroku.com/apps/${your-app-name}` にいく
2. suffix に `/app-json` とつける
3. 以下のような画面が出てくる

   ![/app-json](https://i.gyazo.com/2bb97fe2f8bdd032308de4595ba0178f.png)

4. 最下部の `output` に app.json が表示される

   ![app.json on output section](https://i.gyazo.com/40519d3c64825d329aa48acfdb454a63.png)

# Context

仕事やプライベートでも [Heroku](https://jp.heroku.com/) をよく使うのが、環境構築を楽にするために README.md に Heroku Deploy Button を作成することが多々ある。
もともと全てスクラッチで `app.json` を作成したり、一度作成したプロジェクトのものをコピー&ペースト&修正したりしていた。
そんなとき、弊社の技術顧問が `/app-json` てダッシュボードの URL につけると` app.json` すぐ作れますよ。と耳寄り情報を享受した。
ちなみに GUI の導線は Review App のどこかからでしか辿り着けないらしく、謎（設定からは辿り着けない）

# FYI

https://devcenter.heroku.com/ja/articles/app-json-schema

https://devcenter.heroku.com/ja/articles/heroku-button
