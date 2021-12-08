---
title: "Block Kit でゲシュタルト崩壊しないために JSX でブロックを記述する"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Slack", "BlockKit", "JSX", "React", "TypeScript"]
published: true
---

# 初めに

この記事は [Slack Advent Calendar 2021](https://qiita.com/advent-calendar/2021/slack) 12/8 のエントリーです。
最近 SlackApp を開発した際に便利だった jsx-slack のお話です。

# [Slack Block Kit](https://api.slack.com/block-kit) とは

SlackApp（bot）開発で利用できるリッチなビューレイアウトです
旧来は Attachments という複雑に入り組んだ JSON を組み合わせて Slack 投稿のレイアウトを表現していましたが、数年前に Attachments よりも直感的でシンプルに見た目をリッチに表現できる Block Kit が発表されました。
現在、Slack Official では Attachments は非推奨となり Block Kit をなるだけ利用するように開発者に促しています。
また Block Kit は Block Kit Builder という GUI で利用できるレイアウトビュアーを提供しています。こちらはドラッグ&ドロップでブロックのレイアウトを調整でき、即時同期的に Block Kit の JSON も生成されます。
実装前に見た目をざっくり構成したかったり、色々な Block を使って遊んでみたい場合はとても便利なのでぜひ利用してみてください

FYI: [Block Kit Builder を使ってインタラクティブな Slack アプリをプロトタイピングしよう](https://qiita.com/seratch/items/628751be65de9eb23a80)

# 辛いことろ

上記のような背景から SlackApp 開発ではよく Block Kit を使用しています。
しかし、エンドユーザとモーダルやフォームなどを介して対話的な SlackApp などを作る場合、ビューレイヤーの記述量がかなり冗長となります。
その際に、Block Kit の各タイプ別に似たような JSON の記述量が増え流ため、ゲシュタルト崩壊を起こすことが多々見受けられます。
Node.js を利用している場合は、オフィシャルで bolt.js という SDK が配布されており型情報は提供されていますが、それでも JSON の限界を個人的に感じています。

# jsx-slack とは

そこで [jsx-slack](https://github.com/yhatt/jsx-slack) の登場です。
jsx-slack とは JSON で記述する Block Kit を React の JSX のように記述できるライブラリです。
もともと speee という会社のエンジニアである yhatt 氏を筆頭に作成されたものでした。
現在は yhatt 氏のリポジトリに譲与され、新しい Block Kit が発表されると jsx-slack も比較的早目にサポートされています。
最近だと Slack が発表した次世代プラットフォーム向けの Slack CLI で利用できる [Deno ランタイムをサポート](https://deno.com/blog/slack)したりりしてます。
では簡単なレイアウトを生 JSON で記載した場合を見てみましょう。

# 記述例

## 生 JSON で記述する場合

Block Kit をよく使う人は各オブジェクトの `type` をみて大体イメージできると思います。
しかしビューロジックが複雑になるにつれ、再利用性や可読性が損なわれ独自でコンポーネント化を強いられることが予測されます。
次に jsx-slack で記述する場合を見てみましょう。

```json
[
  {
    "type": "section",
    "text": {
      "type": "mrkdwn",
      "text": "Enjoy building blocks!\n\n&gt; *<https://github.com/yhatt/jsx-slack|jsx-slack>*\n&gt; _Build JSON for Slack Block Kit from JSX_\n&gt; ",
      "verbatim": true
    },
    "accessory": {
      "type": "image",
      "image_url": "https://github.com/yhatt.png",
      "alt_text": "yhatt"
    }
  },
  {
    "type": "context",
    "elements": [
      {
        "type": "mrkdwn",
        "text": "Maintained by <https://github.com/yhatt|Yuki Hattori>",
        "verbatim": true
      },
      {
        "type": "image",
        "image_url": "https://github.com/yhatt.png",
        "alt_text": "yhatt"
      }
    ]
  },
  {
    "type": "divider"
  },
  {
    "type": "actions",
    "elements": [
      {
        "type": "button",
        "text": {
          "type": "plain_text",
          "text": "GitHub",
          "emoji": true
        },
        "url": "https://github.com/yhatt/jsx-slack"
      },
      {
        "type": "button",
        "text": {
          "type": "plain_text",
          "text": "npm",
          "emoji": true
        },
        "url": "https://npm.im/jsx-slack"
      }
    ]
  }
]
```

## jsx-slack を利用して JSX 風に記載する場合

生 JSON で記載されたコード量がわずか 20 行程度にシュリンクできました。
また 一般的によく使用される HTML タグがサポートされているため、直感的にビューを記述できます。
実際にサポートされているタグは [REPL](https://jsx-slack.netlify.app/) や `*.jsx`, `*.tsx` の拡張子で記述するとエラーが表示され、すぐに確認できます。

```jsx
<Blocks>
  <Section>
    <p>Enjoy building blocks!</p>
    <blockquote>
      <b>
        <a href="https://github.com/yhatt/jsx-slack">jsx-slack</a>
      </b>
      <br />
      <i>Build JSON for Slack Block Kit from JSX</i>
    </blockquote>
    <img src="https://github.com/yhatt.png" alt="yhatt" />
  </Section>
  <Context>
    Maintained by <a href="https://github.com/yhatt">Yuki Hattori</a>
    <img src="https://github.com/yhatt.png" alt="yhatt" />
  </Context>
  <Divider />
  <Actions>
    <Button url="https://github.com/yhatt/jsx-slack">GitHub</Button>
    <Button url="https://npm.im/jsx-slack">npm</Button>
  </Actions>
</Blocks>
```

## プロジェクトで実際に使用しているコード

[notion-database-crawler](https://github.com/75asa/notion-database-crawler) では Notion のプロパティをパースし、各プロパティタイプ別に Block Kit を生成しています。

```tsx
import { Prisma } from ".prisma/client";
import { isPropertyValue, parsePrismaJsonObject } from "~/utils";
import { Config } from "~/Config";
import JSXSlack, { Field, Section } from "jsx-slack";
import { MultiSelectProperty } from "~/model/valueObject/slack/notion/properties/MultiSelectProperty";
import { DateProperty } from "~/model/valueObject/slack/notion/properties/DateProperty";
import { SelectProperty } from "~/model/valueObject/slack/notion/properties/SelectProperty";

const { VISIBLE_PROPS } = Config.Notion;

interface PropertiesProps {
  properties: Prisma.JsonObject;
}

export const Properties = ({ properties }: PropertiesProps) => {
  const parsed = parsePrismaJsonObject(properties);
  const element: JSXSlack.JSX.Element[] = [];

  for (const VISIBLE_PROP of VISIBLE_PROPS) {
    const parsedProp = parsed.find(p => p.key === VISIBLE_PROP);
    if (!parsedProp) continue;
    const { key, value } = parsedProp;
    if (!isPropertyValue(value)) continue;
    switch (value.type) {
      case "multi_select": {
        element.push(<MultiSelectProperty key={key} property={value} />);
        break;
      }
      case "date": {
        element.push(<DateProperty key={key} property={value} />);
        break;
      }
      case "select": {
        element.push(<SelectProperty key={key} property={value} />);
        break;
      }
      default:
        break;
    }
  }

  if (!element.length) return <></>;

  return (
    <>
      <Section>
        <p>
          <i>Properties</i>
        </p>
        {element.map(item => (
          <Field>{item}</Field>
        ))}
      </Section>
    </>
  );
};
```

以下は Notion の 複数選択プロパティのコンポーネントです。
同じような形で `<DateProperty>` `<SelectProperty>` も表現しています。

```tsx
import { PropertyValueMultiSelect } from "~/@types/notion-api-types";

interface MultiSelectPropertyProps {
  key: string;
  property: PropertyValueMultiSelect;
}

export const MultiSelectProperty = ({
  key,
  property,
}: MultiSelectPropertyProps) => {
  return (
    <>
      <p>
        <b>{key}</b>: {property.multi_select.map(v => v.name).join(", ")}
      </p>
    </>
  );
};
```

また型情報は全てサポートされており、`*.tsx` の場合そのメリットを大いに享受できます。
`*.js` `*.ts` ではテンプレートリテラルでも利用できます。
気軽に実装する場合などはいいかもしれませんがリテラル内で利用する場合は明示的に型情報を指定しないといけないため、おすすめは `*.jsx` `*.tsx` で利用する方法です。

# まとめ

JSON で冗長な Block Kit に辟易しているほうはぜひお試しを！

# FYI

- [yhatt/jsx-slack](https://github.com/yhatt/jsx-slack)
- [jsx-slack: REPL](https://jsx-slack.netlify.app/)
- [実践 jsx-slack: jsx-slack + Bolt で Slack のモーダルを自在に操ろう](https://tech.speee.jp/entry/2019/10/16/100022)
- [jsx-slack が v2 になりました: 注目の更新点をご紹介します](https://tech.speee.jp/entry/2020/05/18/093300)
- [Slack がプラットフォーム再構築、ワークフローやアプリ開発の環境を強化](https://news.yahoo.co.jp/articles/fa97ce714710c216e9cc0fd2a58b04ef178bedbb)
- [Slack Introduces New Platform With Help From Deno](https://deno.com/blog/slack)
- [Bolt + jsx-slack + TypeScript を使って Slack App をつくる](https://qiita.com/phigasui/items/63c066e06a1708628ba7)
- [75asa/notion-database-crawler](https://github.com/75asa/notion-database-crawler)
- [Slack の Block Kit を jsx の記法で書ける jsx-slack を試してみた](https://qiita.com/selmertsx/items/f3dbd2e4274e8a7966a8)
- [jsx-slack のご紹介&実践 Slack アプリ開発](https://speakerdeck.com/yhatt/jsx-slack-falsegoshao-jie-and-shi-jian-slack-apurikai-fa)
