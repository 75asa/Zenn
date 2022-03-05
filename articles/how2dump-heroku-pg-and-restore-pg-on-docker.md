---
title: "Heroku Postgres から最新データをバックアップし、 Docker で動かしてる Postgres にリストアする"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Heroku", "Postgres", "Docker"]
published: false
---

# はじめに

## 前提

ローカルの git remote に Heroku URL が設定されていることとします。
`$ git remote -v` と入力後に `${YOUR_HEROKU_REMOTE_NAME} https://git.heroku.com/{$YOUR_APP_NAME}.git (fetch)` があることを確認してください。

Heroku CLI はインストール済みのものとします。

https://devcenter.heroku.com/ja/articles/heroku-cli

# 手順

## CLI で Heroku のデータをバックアップする

以下のコマンドでリモートの Heroku Postgres からバックアップデータを生成します。

```fish
$ heroku pg:backups:capture --remote heroku-prd
```

Heroku の公式ガイドがわかりやすく記載されているのでオプションなど気になった方はそちらを参考にして下さい。

https://devcenter.heroku.com/ja/articles/heroku-postgres-backups

## ローカルにバックアップデータを保存する

以下のコマンドで Postgres に一時的に公開 URL を設定し、 curl でローカルにダウンロードします。

```fish
$ curl -o latest.dump (heroku pg:backups public-url --remote heroku-prd)
```

公開 URL って推測されたりセキュリティ的に危ないのでは？と思われる方もいると思います。
Heroku 公式によると、推測は困難かつ 60min 後に URL の公開が終了するということです。

FYI: [バックアップをダウンロードする](https://devcenter.heroku.com/ja/articles/heroku-postgres-backups#downloading-your-backups)

## Docker で Postgres にリストアする

以下のコマンドでローカルのバックアップデータを Docker で動かしている Postgres にリストアします。

```fish
$ docker exec -i postgres-notion-database-crawler pg_restore --verbose --clean -U notion --no-acl --no-owner -d notion < latest.dump
```

何やらオプションが多いですね。
それぞれ見てみましょう。

### docker

`docker exec` は 指定した Docker Name のコンテナを実行するコマンドですね。
ここでは、 `postgres-notion-database-crawler` という名前のコンテナを実行しています。
`-i` は標準入力を聞き続けるオプションだそうです。

### pg_restore

`--verbose`

冗長モード

`--clean`

対象のデータベースを作成

`--no-acl`

`--no-owner`

オブジェクトの所有権の復元を省略

`-d`

データを指定

### 完了すると

このように pg_restore のコマンドが表示されます。

![after running command](https://i.gyazo.com/bb405a5d95e1b0f8254fc3d54c86051f.png)



# FYI

https://devcenter.heroku.com/ja/articles/heroku-postgres-import-export

https://qiita.com/TongTheDopeness/items/06fb5026b51459986f2e

https://fullstacklife.net/database/postgresqlpg-backup-and-restore/