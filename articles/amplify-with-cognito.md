---
title: "Amplify で Cognito を切り替える方法"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# Amplify で Cognito を切り替える方法

## TL;DL;

- Cognito だけを先に作る（既存の Cognito から設定したほうが GUI なので便利）
- `$ amplify remove auth` で現 Amplify env の Cognito を削除する
- `$ amplify import auth` で初めに作った Cognito を紐付ける

## 前提

- Amplify は AWS の各種サービスをオールインワンで使いやすくしてるもの
- 何も設定しないと Amplify デフォルトのままよしなにしてくれる

## 注意点

- `aws-exports.js` は各環境ごとに自動生成されるが、何も設定しないとサーバ上の Amplify Admin での設定が上書きされてしまう

## Cognito を使う

- `$ amplify add auth` でデフォルトの Cognito を追加できる
- 注意点としては、Cognito のユーザ名方針（ユーザネームかアドレスや電話番号どれを使うか）を間違えると作り直すしかなくなる
- 再度作り直したい際は、一度 `$ amplify remove auth` をする
  - この時 Cognito に依存するサービス e.g. API, Function がある場合、あらかじめ依存性を削除するかサービス自体を削除しないといけない
  - 削除するとローカルのディレクトリと template json がごっそりなくなるので、一時的に CognitoUserPool を切り替えたいだけとうときは、依存性の削除だけをおすすめする

### 既存の Cognito を使う場合

- `$ amplify import auth` を実行し、CLI から UserPool もしくは IdentifyProvider を選ぶ

### 新規で Cognito を作り直す場合

- `$ amplify add auth` で作り直し
- p.s. amplify CLI v6 からはユーザネーム方針をどうするか選べるようになったそう
