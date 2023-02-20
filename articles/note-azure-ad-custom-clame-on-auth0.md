---
title: "Auth0 で Azure AD のカスタムクレームを取得するときに気をつけたい"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Azure", "auth0", "OIDC", "oauth2"]
published: true
---

# はじめに

ことの始まりは、 Azure AD で SSO するために Auth0 を使おうとした際、 Azure AD のカスタムクレームが取得できないことに気づいたことから始まります。

Azure AD の SSO を Auth0 で実現するには大まかに分けて以下のように 4 つの方法があります。

1. Microsoft Azure AD
1. OpenID Connect (OIDC)
1. SAML2
1. WS-FED

https://share.cleanshot.com/lQhcysfr

今回の要件は SPA の構成のため、 #1 の Microsoft Azure AD か #2 の OpenID Connect (OIDC) を検討しました。

筆者は手始めに Azure AD のロゴが表示されている #1 の Microsoft Azure AD を採用しました。
これは検索で出てくる記事が #1 の Microsoft Azure AD を採用していることが多かったためです。

筆者が検索した際は ClassMethod さんの記事がトップに表示されていました。

https://dev.classmethod.jp/articles/set-up-enterprise-connection-and-integrate-authentication-of-auth0-app-into-azure-active-directory/

ありがたいことに上記の記事通りに設定することで、 SPA との疎通も無事に完了することができます。
これでログイン基盤はもう触ることはないだろうと思っていました。

# 新たな要望

しばらくすると、 Azure AD のカスタム属性を SPA で表示することができないかと要望が来ました。
具体例を挙げるなら Profile の部署や役職ですね。

Auth0 では Users Management API が提供されているため、 Azure AD の SSO をしたユーザのプロフィールは Auth0 の User Pool で確認することが出来ます。

そこで実際の値を確認したところ、カスタム属性は取得できていませんでした。
取得できているのはユーザの本名、メールアドレス、ユーザ名のみでした。

そこで色々と調査をすると、 Auth0 Action を使って `group` を取得できるという記事を見つけました。

https://community.auth0.com/t/adding-a-rule-to-return-azure-ad-groups-for-a-user-logging-in/56835

そこでカスタム属性を Action の引数から探すことにしました。
しかし、いくら探しても見つかりません。

> Action での取得はできないのか？

とも思ったので一度公式ドキュメントに原点回帰することにしました。

https://auth0.com/docs/authenticate/identity-providers/enterprise-identity-providers/azure-active-directory/v2

これまでざっくり見て理解した気になっていたのですが Note に以下のことが書いていました。

> Claims returned from the Azure AD enterprise connection are static; custom or optional claims will not appear in user profiles. If you need to include custom or optional claims in user profiles, use a SAML or OIDC connection instead.

Auth0 デフォルトの Azure AD ではカスタム属性を取得することができないということですね。
代わりに OIDC で設定するようにとのことです。

# 結論

Azure AD のカスタム属性を SPA ベースで取得するには、 OIDC を使う必要があります。

OIDC の設定手順は以下のコミュニティ記事が参考になります。
実際にこの通りに設定することで、カスタム属性を取得することができました。

https://community.auth0.com/t/connect-to-azure-ad-using-an-oidc-enterprise-connection/55681

同じように困った方の目に留まれば幸いです。

## P.S.

まずはドキュメントを読んで全て理解しよう。

# FYI

https://auth0.com/docs/authenticate/identity-providers/enterprise-identity-providers/azure-active-directory/v2

https://community.auth0.com/t/connect-to-azure-ad-using-an-oidc-enterprise-connection/55681
