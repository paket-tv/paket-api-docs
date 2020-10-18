# Authentication

Paket employs user access tokens to allow access to the API. Access tokens are issued to the requesting client on successful user authentication using the Paket OAuth2 authorization code grant flow:

- Construct the login url or use the AUTHORIZATION Endpoint to render the Paket login page.  
- On successful authentication, Paket will return the `authorization_code`.  
- Use the TOKEN Endpoint to exchange the `authorization_code` for the `access_token`, `id_token`, and `refresh_token` or to request a new `access_token` using the `refresh_token`  
- Include the `access_token` in the `Authorization` header when making calls to the Paket API endpoints. 

<aside class="notice">
  For authorizing users in native apps and in keeping with OAuth2 <a href='https://www.rfc-editor.org/rfc/rfc8252.txt' target='_blank'>best practices</a>, please perform the authorization request in an external user agent (i.e., browser) rather than an embedded user agent.
</aside>

<aside class="notice">
  While not officially tested, the authorization flow should work with most framework-specific Oauth2 clients 
</aside>

<aside class="notice">
  In most cases, this documentation will work with your existing input-restricted device flows. If needed, Paket offers its own code-based flow for input-restricted devices. Please contact us for more information.
</aside>

## AUTHORIZATION Endpoint

```shell
curl GET 'https://auth.paket.tv/oauth2/authorize?response_type=code
&client_id=<client_id>&redirect_uri=<redirect_uri>&code_challenge_method=S256
&code_challenge=<code_challenge>'
```

> The above example uses PKCE and returns the following redirect URL:  
> <span style="color: green">`302 Found`</span>

```
Location: https://<redirect_uri>?code=<authorization_code>
```

Use the AUTHORIZATION Endpoint to render the Paket hosted sign-in form for the requesting client and receive an  
`authorization_code`.

### HTTP Request

`GET https://auth.paket.tv/oauth2/authorize`

### Request Parameters

Parameter | Type | Default | Description
--------- | ------- | ------ | -----------
`response_type` | <small>string</small> | `code` | The response type for the authorization code flow (`code`)
`client_id` | <small>string</small> | | The requesting client's ID as specified in the publisher portal
`redirect_uri` | <small>string</small> | | The client Callback URL as specified in the publisher portal
`code_challenge_method` | <small>string</small> | `S256` | The method used to generate the challenge when using PKCE (supports only `S256`)
`code_challenge` | <small>string</small>  | | Required only when the code_challenge_method is specified.

## TOKEN Endpoint

```shell
curl --location --request POST 'https://auth.paket.tv/oauth2/token&grant_type=authorization_code
&client_id=<client_id>&code=<authorization_code>&redirect_uri=<redirect_uri>' \
--header 'Content-Type: "application/x-www-form-urlencoded"'
```

> The above example uses the `code` grant type and returns JSON structured like this:  
> <span style="color: green">`200 OK`</span>

```json
{ 
  "access_token": "...eyJz9sdfsdfsdfsd", 
  "refresh_token": "...dn43ud8uj32nk2je", 
  "id_token": "...dmcxd329ujdmkemkd349r",
  "token_type": "Bearer", 
  "expires_in": 3600
}
```

Use the TOKEN Endpoint to exchange and `authorization_code` or `refresh_token` for an `access_token`.

### HTTP Request

`POST https://auth.paket.tv/oauth2/token`

### Request Headers

Parameter | Type | Value
--------- | ------- | ------
`Content-Type` | <small>string</small> | `application/x-www-form-urlencoded` 

### Request Parameters

Parameter | Type | Description
--------- | ------- | ------ 
`grant_type` | <small>string</small> | Grant type. Must be `authorization_code` or `refresh_token` <small>[1]</small>
`client_id` | <small>string</small> | The requesting client's ID as specified in the publisher portal
`redirect_uri` | <small>string</small> | Must be the same `redirect_uri` that was used to get `authorization_code` in /oauth2/authorize. Required only if `grant_type` is `authorization_code`
`refresh_token` | <small>string</small> | The `refresh_token`. Required if `grant_type` is `refresh_token`
`code` | <small>string</small> | The `authorization_code`. Required if `grant_type` is `authorization_code`
`code_verifier` | <small>string</small> | The proof key. Required if `grant_type` is `authorization_code` and the authorization code was requested with PKCE.

<small>[1] The TOKEN Endpoint returns a `refresh_token` only when the `grant_type` is `authorization_code`</small> 

## LOGOUT Endpoint

```shell
curl GET 'https://auth.paket.tv/logout?client_id=<client_id>
&logout_uri=<logout_uri>'
```

Use the LOGOUT Endpoint to log user out of the application client.

### HTTP Request

`GET https://auth.paket.tv/logout`


### Request Parameters

Parameter | Type | Description
--------- | ------- | ------ 
`client_id` | <small>string</small> | The requesting client's ID as specified in the publisher portal
`logout_uri` | <small>string</small> | Optional. The client Deauthorize Callback URL as specified in the publisher portal
