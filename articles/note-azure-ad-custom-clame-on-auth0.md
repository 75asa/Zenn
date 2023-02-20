---
title: "Auth0 で Azure AD のカスタムクレームを取得するときに気をつけたい"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Azure", "auth0", "OIDC", "oauth2"]
published: false
---

# はじめに

ことの始まりは、 Azure AD で SSO するために Auth0 を使おうとした際、 Azure AD のカスタムクレームが取得できないことに気づいたことから始まります。

> Claims returned from the Azure AD enterprise connection are static; custom or optional claims will not appear in user profiles. If you need to include custom or optional claims in user profiles, use a SAML or OIDC connection instead.

# FYI

https://auth0.com/docs/authenticate/identity-providers/enterprise-identity-providers/azure-active-directory/v2