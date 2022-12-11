---
title: "VSCode と GitHub Action を使って typo を滅する"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typo", "cspell", "github", "action", "vscode"]
published: false
---

## PR のレビューで typo 指摘するのめんどくさくないですか？

筆者はコードレビューをする際に、 typo を指摘するのがずっとめんどくさいと思ってました。
そこで、CSpell の [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker) を普段使っている VSCode に入れています。
これで自分の typo は防げていたのですが、チーム開発では個人の努力だけではどうにもなりません。
また、チーム規模が大きくなるにつれてコントロールできず typo の波線がずっと表示されるなんてこともしばしば...。
そこで今回は以下のような仕組み化で解決するようにしました。

1. CI に typo 検知のワークフローを入れる
2. VSCode を使っている開発者が辞書登録をしやすいようにする

それでは設定方法を見てみましょう。

## VSCode Extension の導入

先ずは VSCode の拡張機能である Code Spell Checker を入れます。

https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker

基本的な使い方は割愛しますが、以下のように typo を検知してくれます。

![detected a typo on VSCode](https://i.gyazo.com/eb15ea1b01c69f3df9a25653102d219b.png)

Quick Fix で辞書登録機能も提供しています。
![Quick Fix Preview](https://i.gyazo.com/147258c5489873013045401e1ac14e8a.png)

またチーム開発でまだこの拡張機能を入れてないユーザ向けに推奨拡張機能として cSpell を入れておきます。

VSCode で Code Spell Checker を開き、 ⌘ Shift P でコマンドパレットを開きます。
`Extensions: Add Extension to Workspace Folder Recommendations` を選択することでプロジェクトルートに `.vscode/extensions.json` が生成されます。

![how to add recommend extension for spell checker](https://i.gyazo.com/b30d78d877a6090d81a8bbaadaf43c52.png)

内容としては以下の json です。

```json
{
  "recommendations": ["streetsidesoftware.code-spell-checker"]
}
```

詳細はこちらの記事が分かりやすいので参考にしてください。

https://future-architect.github.io/articles/20200828/

## package の導入

次に cSpell package を入れます。
`$ yarn add -D cspell` でインストールします。

## cSpell 初期設定

### cSpell 用ディレクトリの生成

project root に .cspell.json を作成する方法でも良いのですが、 cSpell の設定ファイルをまとめておきたいので今回は以下のようにプロジェクトルートにディレクトリを作成します。
`$ mkdir .cspell`

### config file の生成

次に cSpell の config ファイルを生成します。
先ほど生成したディレクトリに `$ touch .cspell/cspell.cjs` でファイルを作成します。
`*.json` でもいいのですが、 cSpell は型情報を内包しているので型情報を活用することをお勧めします。
またコメントなども書けるのも良いですね。

config ファイルは以下のようにします。
他にも様々なオプションがありますがここでは割愛します。
詳細は[公式ドキュメント](http://cspell.org/)を参照してください。

```js
/** @type { import("@cspell/cspell-types").CSpellUserSettings } */
module.exports = {
  $schema:
    "https://raw.githubusercontent.com/streetsidesoftware/cspell/main/cspell.schema.json",
  version: "0.2",
  // dictionaryDefinitions[number].name と同じ名前を指定する
  dictionaries: ["words"],
  // カスタム辞書ファイル配列、複数指定可能
  dictionaryDefinitions: [
    // 辞書ファイル
    {
      name: "words",
      path: "./.cspell/words.txt",
      addWords: true,
    },
  ],
  // cSpell 除外設定
  ignorePaths: ["./words.txt"],
};
```

コメントにもあるように辞書ファイルを設定しています。
今回は一つの `./.cspell/words.txt` だけですが、複数の辞書ファイルを設定することも可能です。

以下ポイントだけ押さえておきます。

- dictionaries
  - dictionaryDefinitions[number].name と同じ名前を指定する
- dictionaryDefinitions
  - 辞書ファイル
  - 拡張子は `.txt`
  - addWords: true で辞書ファイルに単語を追加できるようになる
- ignorePaths
  - cSpell でチェックしないパスを指定する

**_最後に cspell.cjs で設定した辞書ファイルを `$ touch .cspell/words.txt` で生成します。_**

### npm script の設定

`"spell-check": "cspell lint './**/*' -c ./.cspell/cspell.cjs --gitignore --show-suggestions"` を npm scripts に設定します。

`--show-suggestions` は typo の候補を表示するオプションです。

`--gitignore` は gitignore に記載されているファイルを除外するオプションです。

試しに README に typo を入れて `$ yarn spell-check`　を実行してみます。
![detect a typo](https://i.gyazo.com/8fc60f04d496168d3fb516f0e2679b31.png)

`Setuup` が typo として検知されました。
`--show-suggestions` で typo の候補が表示されていますね。
今回だと `[setup, stepup, setula, setups, setout]` です。

これで npm script でタイポを検知できるようになりました。
GitHub Actions で検知するのが金銭的にしんどい場合であれば husky などでカバーするのお勧めです。
以下の記事で詳しく解説されています。

https://zenn.dev/luvmini511/articles/ade1f0e4b64770

さて次は VSCode の設定を見直します。

## VSCode での cspell の設定

VSCode で気軽に共通の辞書ファイルへの登録ができるように `.vscode/settings.json` を生成し、以下の項目を追加します。
この内容は `.cspell/cspell.cjs` の customDictionaryDefinitions と同じです。
二度手間になるのは気になりますが、そこまで頻繁に更新するものではないのでまだ良いかと感じています。
もし、共通管理できる方法をご存知の方がいましたら教えていただけると嬉しいです。

```json
{
  "cSpell.customDictionaries": {
    "words": {
      "name": "words",
      "path": "${workspaceRoot}/.cspell/words.txt",
      "description": "Words used in this project",
      "addWords": true
    },
    "custom": true, // Enable the `custom` dictionary
    "internal-terms": false // Disable the `internal-terms` dictionary
  }
}
```

これにより VSCode の Quick Fix でカスタム辞書に単語を追加できるようになります。

![Quick Fix to adding the customized dict a word](https://i.gyazo.com/9ea01b6f094d118506abbbbf716eeab0.png)

## GitHub Actions での cspell の設定

最後は、 GitHub Actions を使って CI でタイポを検知する設定を行います。
`$ mkdir -p .github/workflows` で `.github/workflows` ディレクトリを作成し、`$ touch .github/workflows/spell-check.yaml` で GitHub Actions の設定ファイルを作成します。
また、 typo を検知した際に reviewdog でファイルにコメントを残すようにしています。
reviewdog の詳細は割愛しますが、以下の記事が参考になります。

https://zenn.dev/peraichi_blog/articles/01fy360dgteynbfv5tj3q6smv5

### GitHub Actions の設定

この設定では、コミットの修正範囲のファイルのみに対象を絞っています。
こうすることで、既存プロジェクトに徐々に導入することができます。

```yaml
name: spell-check

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  spell-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - uses: reviewdog/action-setup@v1
        with:
          reviewdog_version: latest
      - uses: actions/setup-node@v2
        with:
          node-version: "18.x"
      - name: Install dependencies
        run: yarn --frozen-lockfile
        shell: bash
      - uses: reviewdog/action-setup@v1
        with:
          reviewdog_version: latest
      - name: execute spell-check
        env:
          REVIEWDOG_GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          git diff remotes/origin/$GITHUB_BASE_REF --name-only \
            | yarn spell-check \
            | reviewdog -efm="%f:%l:%c - %m" -reporter=github-pr-check
```

### 実際の挙動

以下に reviewdog でコメントされた例を載せておきます。

https://github.com/75asa/cSpell-template/pull/1

CI の設定でエラーと判定することも可能ですが、今回はしていません。
#1 で Revewdog でコメントされた typo をあえて無視して次の PR を作成してみました。

https://github.com/75asa/cSpell-template/pull/3

ここでは package.json のみを修正しているため、 #1 でコメントされた typo は無視されています。

## まとめ

コードレビューで typo を指摘するのがめんどくさい方、またはすでに typo だらけで徐々に改善したいと思っている方におすすめです。

## FYI

https://t-yng.jp/post/spell-check-ci
https://dev-yakuza.posstree.com/linter/cspell/
