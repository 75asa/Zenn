---
title: "Service Now Table API で複数の sys_id を sysparm_query に入れてリクエストを送る際の最大許容数"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ServiceNow", "TableAPI"]
published: false
---

## TL;DR

- sys_id の桁数は 32 文字固定
- 171 レコードは問題ない
- 他の query がなければ URL の text length は 9084
- URL 許容文字数を越えた場合は以下のような 414 エラーが表示される

```html
<html>
  <head>
    <title>414 Request-URI Too Large</title>
  </head>
  <body>
    <center><h1>414 Request-URI Too Large</h1></center>
    <hr />
    <center>snow_adc</center>
  </body>
</html>
```

以下のようにする

```shell
curl 'https://{workspace-name}.service-now.com/api/now/table/{target-table}?&sysparm_query=sys_idLIKE{sys_id_A}sys_idLIKE{sys_id_B}'
```

## FYI

- https://developer.servicenow.com/dev.do#!/reference/api/tokyo/rest/c_TableAPI#table-GET
