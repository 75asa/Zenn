---
title: "Azure AD と Auth0 を使ってログインした際に Azure AD から Microsoft Graph API を使う"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## 前提

- Azure AD を IdP として Auth0 に設定している
- OIDC ではなく Microsoft Identify API v2 を使っている

## やりたかったこと

- Auth0 を使って Azure AD でログインした際に、 Microsoft Graph API を使ってエンドユーザがサインインしているデバイス情報を取得したい

## やり方

### Azure App にて利用したい API を許可する

### connection の設定を確認する

### connection の設定で scope を更新する

### User Identities にて、 Azure AD の Access Token が含まれているか確認する

### Microsoft Graph API を使って、エンドユーザがサインインしているデバイス情報を取得する

## FYI

- https://community.auth0.com/t/auth0-token-refresh-does-not-automatically-refresh-azure-ad-issued-access-token/88927
- https://learn.microsoft.com/ja-jp/graph/api/user-list-owneddevices?view=graph-rest-beta&tabs=http
