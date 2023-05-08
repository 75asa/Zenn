---
title: "Azure AD と Auth0 を使ってログインした際に Azure AD から Microsoft Graph API を使う"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["auth0", "msgraphapi", "oidc", "azure", "microsoft"]
published: true
---

## 前提

- Azure AD を IdP として Auth0 に設定している
- OIDC ではなく Microsoft Identify API v2 を使っている
- Azure App を作成している

## やりたかったこと

Auth0 を使って Azure AD でログインした際に、 Microsoft Graph API を使ってエンドユーザがサインインしているデバイス情報を取得することが目的でした。
しかし、 Auth0 ダッシュボードでは、 Azure AD 側の scope を設定することはできません。
ここの scope は OIDC 準拠なため、 Azure AD などの 3rd party 独自のスコープが設定できないようです。

> There is no standard way to renew identity provider access tokens through Auth0. The mechanism for renewing identity provider access tokens varies for each provider. In the case of Azure AD, you can request refresh token to renew the access token once it expires. Auth0 dashboard does not allow you to include refresh token scope when autneticating with Azure AD. However, one can set scope manually via Management API and request additional scope.

https://gist.github.com/Tanver-Hasan/640ab259a333151df35c8efe9cbc839f

なんとかやり方がないかと調べたところ Auth0 Community にて有益な情報があったので、ドキュメントに残しておきます。

## やり方

### Azure App にて利用したい API を許可する

先ずは Auth0 の接続時に作成した Azure App の API Permissions にて、利用したい API を許可します。

今回は以下の 2 つの API を使いたいのでアクセス許可対象の `Device.Read.All` と `Directory.Read.All` を許可します。

https://learn.microsoft.com/ja-jp/graph/api/device-list?view=graph-rest-1.0&tabs=http

https://learn.microsoft.com/ja-jp/graph/api/user-list-owneddevices?view=graph-rest-1.0&tabs=http

以下のキャプチャでは、 Microsoft Graph API の `Directory.Read.All` を許可しています。
同じように `Device.Read.All` も許可してください。

[Azure Apps API Permissions](https://user-images.githubusercontent.com/38843176/102493673-6da2db80-406b-11eb-9e87-1f45dd88d06a.png)

### NOTE

この際に、変更した権限が Grant されて緑色になっていることを確認してください。
管理者アカウントではない場合は管理者の承認が必要になるため、 Azure AD の管理者に承認を依頼してください。

## Azure App にて Authentication の implicit grant Access tokens を許可する

次に、 Azure App の Authentication にて implicit grant の Access tokens を許可します。
これにより Access token をログインフローのペイロードに含めることが出来ます。

![implicit include access token figure](https://i.gyazo.com/cd4f26452da8bb037940585e478408cd.png)

### connection の設定を確認する

次に Auth0 で設定している Azure AD の connection の内容を確認します。

以下のエンドポイントで connection id を引数にして確認できます。
auth0 ダッシュボードで id を確認するには Authentication -> Enterprise に行き接続 Application を確認します。
タイトルのすぐ下に con\_\*\*\*\* となっているのが connection ID です。

https://auth0.com/docs/api/management/v2#!/Connections/get_connections_by_id

Bearer token には Management API で取得した token を設定してください。

```shell
curl -X GET \
  'https://{auth0domain}.us.auth0.com/api/v2/connections/con_***' \
  --header 'Authorization: Bearer ${your_bearer_token}'
```

戻り値のサンプルとしては以下のようなペイロードが返却されます。

```json
{
  "name": "My connection",
  "display_name": "",
  "options": {},
  "id": "con_0000000000000001",
  "strategy": "waad",
  "realms": [""],
  "enabled_clients": ["avUAvH1pgnZGgAGlv8guZLPoaOnjVJsM"],
  "is_domain_connection": false,
  "metadata": {}
}
```

### connection の設定で scope を更新する

次にこの Connection の設定を更新します。
`options` に `upstream_params` という項目を追加します。
今回は利用したい API の scope を設定するため、以下のように設定します。

```json
    "upstream_params": {
      "scope": {
        "value": "openid profile email User.Read Directory.Read.All"
      }
    },
```

これは Dynamic parameters と呼ばれ、 IdP に渡すパラメータを設定することができます。

Dynamic parameters についの詳細は以下のドキュメントを参照してください。

https://auth0.com/docs/authenticate/identity-providers/pass-parameters-to-idps#dynamic-parameters

更新の方法は以下のエンドポイントを利用します。
https://auth0.com/docs/api/management/v2?_ga=2.224570586.138144054.1682643450-1705404137.1673511033&_gl=1*1pstcix*rollup_ga*MTcwNTQwNDEzNy4xNjczNTExMDMz*rollup_ga_F1G3E656YZ*MTY4MjY0MzQ0OS4zNS4xLjE2ODI2NDM4NzQuNjAuMC4w#!/Connections/patch_connections_by_id

#### NOTE

設定の更新は破壊的な変更のため、実行する前に必ず get connection で取得した内容を保存しておいてください。

### User Identities にて、 Azure AD の Access Token が含まれているか確認する

では実際にログインしてみましょう。
初めてログインした際、 Microsoft のログイン画面にて今回変更した scope の許可を求められます。
こちらを許可すると、 Auth0 の User Identities にて Azure AD の Access Token が含まれていることが確認できます。
なお、こちらはクライアント側の getUsers では確認できません。
サーバーサイド、もしくは API で確認してください。

### Microsoft Graph API を使って、エンドユーザがサインインしているデバイス情報を取得する

Microsoft Graph API を使って、エンドユーザがサインインしているデバイス情報を取得してみましょう。

oid は waad 経由で登録された Auth0 User のプロフィール項目にあります。
MS 側で利用しているオリジンの ID です。
token には 前述の MS 側の Access Token を利用してください。

```shell
curl -X GET \
  'https://graph.microsoft.com/v1.0/users/${auth0-user-oid}/ownedDevices' \
  --header 'Authorization: Bearer ${token}'
```

レスポンスのサンプルとしては以下のようなペイロードが返却されます。
デバイス名やデバイス ID などが取得できます。
ユースケースに応じて適宜利用してください。

```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#directoryObjects",
  "value": [
    {
      "@odata.type": "#microsoft.graph.device",
      "id": "****",
      "deletedDateTime": null,
      "accountEnabled": true,
      "approximateLastSignInDateTime": "2022-05-27T05:30:32Z",
      "complianceExpirationDateTime": null,
      "createdDateTime": "2022-05-11T09:06:17Z",
      "deviceCategory": null,
      "deviceId": "****",
      "deviceMetadata": null,
      "deviceOwnership": null,
      "deviceVersion": 2,
      "displayName": "****",
      "domainName": null,
      "enrollmentProfileName": null,
      "enrollmentType": null,
      "externalSourceName": null,
      "isCompliant": null,
      "isManaged": null,
      "isRooted": null,
      "managementType": null,
      "manufacturer": null,
      "mdmAppId": null,
      "model": null,
      "onPremisesLastSyncDateTime": null,
      "onPremisesSyncEnabled": null,
      "operatingSystem": "Windows",
      "operatingSystemVersion": "****",
      "physicalIds": ["****", "****", "****", "****"],
      "profileType": "****",
      "registrationDateTime": "2022-05-11T09:06:17Z",
      "sourceType": null,
      "systemLabels": [],
      "trustType": "****",
      "extensionAttributes": {
        "extensionAttribute1": null,
        "extensionAttribute2": null,
        "extensionAttribute3": null,
        "extensionAttribute4": null,
        "extensionAttribute5": null,
        "extensionAttribute6": null,
        "extensionAttribute7": null,
        "extensionAttribute8": null,
        "extensionAttribute9": null,
        "extensionAttribute10": null,
        "extensionAttribute11": null,
        "extensionAttribute12": null,
        "extensionAttribute13": null,
        "extensionAttribute14": null,
        "extensionAttribute15": null
      },
      "alternativeSecurityIds": [
        {
          "type": 2,
          "identityProvider": null,
          "key": "***"
        }
      ]
    }
  ]
}
```

## まとめ

ユースケースとしてはよくありそうですが筆者が探していた際は、日本語の記事が見つからなかったのでここに備忘録として残しました。
個人的には Auth0 を使わずに MS Graph SDK や自前で実装するケースの方が多い気がしています。
ただ Auth0 を経由することにより MS 以外の ID プロバイダを利用する際にも同じようなことができるので、 Google や Facebook などの ID プロバイダを利用する際にも参考になるかと思います。

## FYI

- https://community.auth0.com/t/auth0-token-refresh-does-not-automatically-refresh-azure-ad-issued-access-token/88927
- https://learn.microsoft.com/ja-jp/graph/api/user-list-owneddevices?view=graph-rest-beta&tabs=http
- https://learn.microsoft.com/ja-jp/graph/api/user-list-owneddevices?view=graph-rest-1.0&tabs=http
