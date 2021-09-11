---
title: "Notion API のデータベースでフィルタする方法"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Notion", "NotionAPI", "TypeScript", "Node"]
published: false
---

# tl;dr;

![](https://i.gyazo.com/c1cafd90401813804350f69e8799c224.png)


- body.filter.property にフィルタリングしたい _DB プロパティ名_ を指定
- 上記の項目のタイプによって 型 を指定
  - Text
    - `title`
    - `rich_text`
    - `url`
    - `email`
    - `phone`
  - Number
    - `number`
  - Checkbox
    - `checkbox`
  - Select
    - `select`
  - Multi-select
    - `multi_select`
  - Date
    - `date`
    - `created_time`
    - `last_edited_time`
  - People
    - `people`
    - `created_by`
    - `lsst_edited_by`
  - Files
    - `files`
  - Relation
    - `relation`
  - Formula
    - `formula`
  - Compound
    - `or`
    - `and`

```typescript
const requestPayload: RequestParameters = {
  path: `databases/${databaseId}/query`,
  method: "post",
  body: {
    // 前回同期した時間以降にフィルター、時間がない場合は現在時刻
    filter: {
      property: "createdAt",
      created_time: {
        on_or_after: lastFetchedAt,
        // on_or_after: "2021-07-08T06:31:00.000Z", // NOTE: for test
      },
    },
  },
};
if (cursor) requestPayload.body = { start_cursor: cursor };
// While there are more pages left in the query, get pages from the database.
let pages = null;
try {
  pages = (await notion.request(requestPayload)) as DatabasesQueryResponse;
  // pages as DatabasesQueryResponse;
} catch (e) {
  console.error(e);
  throw e;
}
```

# FYI

- [Notion API Database Filter](https://developers.notion.com/reference/post-database-query#post-database-query-filter)
