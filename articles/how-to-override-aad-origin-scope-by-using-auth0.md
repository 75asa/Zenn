---
title: "Auth0 経由で Azure AD の scope を更新する"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# あらまし

仕事で Auth0 を使って Azure AD の SSO を実現していた際に、 Azure AD の scope を更新する必要がありました。
しかし、デフォルトでは scope は変更できないようで、どうすればいいのかわからなかったので調べてみました。
そこで Auth0 のコミュニティを探したところやり方が見つかったのでドキュメントに残しておこうと思います。

