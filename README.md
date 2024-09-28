# Traefik OpenID Connect Middleware

<p align="left" style="text-align:left;">
  <a href="https://github.com/sevensolutions/traefik-oidc-auth">
    <img alt="Logo" src=".assets/icon.png" width="150" />
  </a>
</p>

A traefik Plugin for securing the upstream service with OpenID Connect acting as a relying party.

> [!NOTE]  
> This document always represents the latest version, which may not have been released yet.  
> Therefore, some features may not be available currently but will be available soon.
> You can use the GIT-Tags to check individual versions.

> [!WARNING]
> This middleware is under development and only tested against [ZITADEL](https://zitadel.com/) yet. Although it should be compatible with any OIDC-compatible IDP.

## 💡 Getting Started

Enable the plugin in your traefik configuration.

```yml
experimental:
  plugins:
    traefik-oidc-auth:
      moduleName: "github.com/sevensolutions/traefik-oidc-auth"
      version: "v0.2.0"
```

Add a middleware and reference it in a route.

```yml
http:
  services:
    whoami:
      loadBalancer:
        servers:
          - url: http://whoami:80

  middlewares:
    oidc-auth:
      plugin:
        traefik-oidc-auth:
          Provider:
            Url: "https://..."
            ClientId: "<YourClientId>"
            ClientSecret: "<YourClientSecret>"
            #UsePkce: true # Or use PKCE instead of a secret
          Scopes: ["openid", "profile", "email"]
          Authorization:
            AssertClaims:
              - Name: "preferred_username"
                Values: "alice@gmail.com,bob@gmail.com"
          Headers:
            MapClaims:
              - Claim: "preferred_username"
                Header: "X-Oidc-Username"
              - Claim: "sub"
                Header: "X-Oidc-Subject"
  
  routers:
    whoami:
      entryPoints: ["web"]
      rule: "HostRegexp(`.+`)"
      service: whoami
      middlewares: ["oidc-auth"]
```

## 🛠 Configuration Options

### Plugin Config Block

| Name | Required | Type | Default | Description |
|---|---|---|---|---|
| Secret | no | `string` | `MLFs4TT99kOOq8h3UAVRtYoCTDYXiRcZ`| A secret used for encryption. Must be a 32 character string. |
| Provider | yes | `Provider` | *none* | Identity Provider Configuration. See *Provider* block. |
| Scopes | no | `string[]` | `["openid", "profile", "email"]` | A list of scopes to request from the IDP. |
| CallbackUri | no | `string` | `/oidc/callback` | Defines the callback url used by the IDP. This needs to be registered in your IDP. |
| LoginUri | no | `string` | *none* | An optional url, which should trigger the login-flow. By default every url triggers a login-flow, if the user is not already logged in. If you set this to eg. `/login`, only this url will trigger a login-flow while all other requests return *Unauthorized*.  |
| PostLoginRedirectUri | no | `string` | *none* | An optional static redirect url where the user should be redirected after login. By default the user will be redirected to the url which triggered the login-flow. |
| LogoutUri | no | `string` | `/logout` | The url which should trigger a logout-flow. |
| PostLogoutRedirectUri | no | `string` | `/` | The url where the user should be redirected after logout. |
| Authorization | no | `Authorization` | *none* | Authorization Configuration. See *Authorization* block. |
| Headers | no | `Headers` | *none* | Header Configuration. See *Headers* block. |

### Provider Block

| Name | Required | Type | Default | Description |
|---|---|---|---|---|
| Url | no | `string` | *none* | The full URL of the Identity Provider. This is required, if *UrlEnv* is not used. |
| UrlEnv | no | `string` | *none* | The name of an environment variable, containing the full URL of the Identity Provider. This is required, if *Url* is not used. |
| ClientId | no | `string` | *none* | The client id of the application. This is required, if *ClientIdEnv* is not used. |
| ClientIdEnv | no | `string` | *none* | The name of an environment variable, containing the client id. This is required, if *ClientId* is not used. |
| ClientSecret | no | `string` | *none* | The client secret of the application. This is required, if *ClientSecretEnv* is not used and *UsePkce* is false. |
| ClientSecretEnv | no | `string` | *none* | The name of an environment variable, containing the client secret. This is required, if *ClientSecret* and *UsePkce* are not used. |
| UsePkce | no | `bool` | `false`| Enable PKCE. In this case, a client secret is not needed. The following algorithms are supported: *RS*, *EC*, *ES*. |
| ValidateIssuer | no | `bool` | `true` | Specifies whether the `iss` claim in the JWT-token should be validated. |
| ValidIssuer | no | `string` | *discovery document* | The issuer which must be present in the JWT-token. By default this will be read from the OIDC discovery document. |
| ValidIssuerEnv | no | `string` | *none* | The name of an environment variable, containing the valid issuer. This is required, if *ValidIssuer* is not used and ValidateIssuer is enabled. |
| ValidateAudience | no | `bool` | `true` | Specifies whether the `aud` claim in the JWT-token should be validated. |
| ValidAudience | no | `string` | *ClientId* | The audience which must be present in the JWT-token. Defaults to the configured client id. |
| ValidAudienceEnv | no | `string` | *none* | The name of an environment variable, containing the valid audience. This is required, if *ValidAudience* is not used and ValidateAudience is enabled. |

### Authorization Block

| Name | Required | Type | Default | Description |
|---|---|---|---|---|
| AssertClaims | no | `ClaimAssertion[]` | *none* | ClaimAssertion Configuration. See *ClaimAssertion* block. |

### ClaimAssertion Block

| Name | Required | Type | Default | Description |
|---|---|---|---|---|
| Name | yes | `string` | *none* | The name of the claim in the access token. |
| Value | no | `string` | *none* | The required value of the claim. If *Value* and *Values* are not set, only the presence of the claim will be checked. |
| Values | no | `string[]` | *none* | An array of allowed strings. The user is authorized if the claim matched any of these. |

### Headers Block

| Name | Required | Type | Default | Description |
|---|---|---|---|---|
| MapClaims | no | `ClaimHeader[]` | *none* | A list of claims which should be mapped as headers when the request will be sent to the upstream. See *ClaimHeader* block. |

### ClaimHeader Block

| Name | Required | Type | Default | Description |
|---|---|---|---|---|
| Claim | yes | `string` | *none* | The name of the claim |
| Header | yes | `string` | *none* | The name of the header which should receive the claim value. |

## 🧪 Local Development

Create the following `.env` file:

```
PROVIDER_URL=...
CLIENT_ID=...
CLIENT_SECRET=...
```

The run `docker compose up` to run traefik locally.

Now browse to http://localhost:9080. You should be redirected to your IDP.  
After you've logged in, you should be redirected back to http://localhost:9080 and see a WHOAMI page.
