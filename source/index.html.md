---
title: Paket API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - javascript
  - ruby
  - python
  - java
  - swift
  - objective_c

toc_footers:
  - <a href='https://publisher.paket.tv'>Sign In to the Publisher Portal</a>
  - <a href='https://publisher.paket.tv/signup'>Request Access</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true
---

# Introduction 

Welcome to Paket. Paket is a utility platform for streaming media services that unifies authentication, billing, and discovery into a seamless and frictionless experience for users.

While nearly all of the Platform's business logic (including account creation, billing, recurring payments, etc.) happens on Paket's end, streaming service providers will need to integrate with Paket's API to be able to authenticate users and to access the authenticated user's entitlements to the respective service via the Paket API.

<aside class="notice">
  While we will be regularly adding business features to the publisher portal prior to launch, we expect few - if any - changes to these client APIs. Any updates or changes will be announced well in advance.
</aside>

# Getting Started

Before we begin, it's important to understand the key concepts and taxonomy of the platform, each of which can be configured in the [Paket Publisher Portal](https://publisher.paket.tv):

**Companies**: The `Company` is the primary object to which Apps are associated. This is where the business details are configured. In most situations, an account will have only one Company;

**Apps**: The `App` is the primary object to which Clients and Licenses are associated. This is the high-level representation of your service, its consumer-facing title (e.g., Hulu), description, and supporting metadata;

**Licenses**: The `License` object represents an entitlement offered via your App (e.g., Limited Ads at $5.99 / mo, No Ads at $9.99 / mo, etc.), its pricing, and regional availability; and

**Clients**: The `Client` object contains the platform or device-specific Paket auth (OAuth2) configuration for a service's user-facing Application client (e.g., Roku, iOS, Android, etc.).

Now that we're familiar with the key concepts and taxonomy, here's how the API works:

1. **Authenticate** the user and retrieve the client-specific `access_token`,`refresh_token`, and `id_token`;
2. **Retrieve** available licenses by calling the `/license` api endpoint and passing the `access_token` in the `Authorization` header; and
3. **Refresh** new access tokens as necessary by calling the `/oauth2/authorization` endpoint using the `refresh_token` response type.

Though the above flow illustrates how to use the Paket API, service providers are encouraged to configure additional logic on their end (e.g., creating a local user or profile object from the OpenID data shared via the `id_token`) so as to deliver the optimal experience to the end user. 

## Statuses 

The following Statuses are assignable to the `App` and `License` models:

Status        | Description
------------- | -----------
Draft | (Default) The default status, assigned on object create. The object is not available to end users.  
Pending | Publisher has deployed the object and is pending Paket approval. The object is not available to end users. 
Live | Publisher has deployed the object and Paket has approved it. The object is available to end users.  
Inactive | Publisher has hidden a Paket-approved object that has active subscribers. The object is not available to new end users.

All `License` objects in test mode whose statuses are "Pending" will be retrieved when calling the API on a test mode client.

## Price Tiers

Paket uses price tiers to ensure pricing consistency and predictability across regions and countries. Pricing and regional availability is assigned to the `License` object. You may create as many licenses as is necessary to ensure that your regional pricing is accurately represented.

<aside class="notice">
  Price tier changes on Live licenses can be scheduled via request to Paket and require a min 30 days lead time.
</aside>

## Testing

Testing on Paket is done in the Production environment using test mode `Client` and `License` objects. This ensures that you are testing your API integration in the very environment to which you will be deploying your App. 

Simply create one or more test mode `Client` and `License` objects within your `App` object by selecting the "Test Mode" switch within the Publisher Portal. This will allow you to test your implementation using the test client. When in test mode, the test client will retrieve all of the App's test mode licenses that are set to "Pending" status. 

When you are ready to deploy your App, simply swap out the test mode `Client` ID for the production-ready one.

# Authentication

Paket employs user access tokens to allow access to the API. Access tokens are issued to the requesting client on successful user authentication using the Paket OAuth2 authorization code grant flow:

- Construct the login url or use the AUTHORIZATION Endpoint to render the Paket login page.  
- On successful authentication, Paket will return the `authorization_code`.  
- Use the TOKEN Endpoint to exchange the `authorization_code` for the `access_token`, `id_token`, and `refresh_token` or to request a new `access_token` using the `refresh_token`  
- Include the `access_token` in the `Authorization` header when making calls to the Paket API endpoints. 

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

```
HTTP/1.1 302 Found
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
`grant_type` | <small>string</small> | Grant type. Must be `authorization_code` or `refresh_token`
`client_id` | <small>string</small> | The requesting client's ID as specified in the publisher portal
`redirect_uri` | <small>string</small> | Must be the same `redirect_uri` that was used to get `authorization_code` in /oauth2/authorize. Required only if `grant_type` is `authorization_code`
`refresh_token` | <small>string</small> | The `refresh_token`. Required if `grant_type` is `refresh_token` <small>[1]</small>
`code` | <small>string</small> | The `authorization_code`. Required if `grant_type` is `authorization_code`
`code_verifier` | <small>string</small> | The proof key. Required if `grant_type` is `authorization_code` and the authorization code was requested with PKCE.

<small>[1] The TOKEN Endpoint returns a `refresh_token` only when the `grant_type` is `authorization_code`</small> 

## LOGOUT Endpoint

### HTTP Request

`GET https://auth.paket.tv/logout`

# Licenses

Endpoints:  

- `GET /license`

## GET /license

```ruby
require "uri"
require "net/http"

url = URI("https://api.paket.tv/license")

https = Net::HTTP.new(url.host, url.port);
https.use_ssl = true

request = Net::HTTP::Get.new(url)
request["Authorization"] = "<access_token>"

response = https.request(request)
puts response.read_body
```

```python
import http.client
import mimetypes
conn = http.client.HTTPSConnection("api.paket.tv")
payload = ''
headers = {
  'Authorization': '<access_token>'
}
conn.request("GET", "/license", payload, headers)
res = conn.getresponse()
data = res.read()
print(data.decode("utf-8"))
```

```shell
curl --location --request GET 'https://api.paket.tv/license' \
--header 'Authorization: <access_token>'
```

```javascript
var myHeaders = new Headers();
myHeaders.append("Authorization", "<access_token>");

var requestOptions = {
  method: 'GET',
  headers: myHeaders,
  redirect: 'follow'
};

fetch("https://api.paket.tv/license", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));
```

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
Request request = new Request.Builder()
  .url("https://api.paket.tv/license")
  .method("GET", null)
  .addHeader("Authorization", "<access_token>")
  .build();
Response response = client.newCall(request).execute();
```

```swift
import Foundation

var semaphore = DispatchSemaphore (value: 0)

var request = URLRequest(url: URL(string: "https://api.paket.tv/license")!,timeoutInterval: Double.infinity)
request.addValue("<access_token>", forHTTPHeaderField: "Authorization")

request.httpMethod = "GET"

let task = URLSession.shared.dataTask(with: request) { data, response, error in 
  guard let data = data else {
    print(String(describing: error))
    return
  }
  print(String(data: data, encoding: .utf8)!)
  semaphore.signal()
}

task.resume()
semaphore.wait()
```

```objective_c
#import <Foundation/Foundation.h>

dispatch_semaphore_t sema = dispatch_semaphore_create(0);

NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"https://api.paket.tv/license"]
  cachePolicy:NSURLRequestUseProtocolCachePolicy
  timeoutInterval:10.0];
NSDictionary *headers = @{
  @"Authorization": @"<access_token>"
};

[request setAllHTTPHeaderFields:headers];

[request setHTTPMethod:@"GET"];

NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request
completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
  if (error) {
    NSLog(@"%@", error);
  } else {
    NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *) response;
    NSError *parseError = nil;
    NSDictionary *responseDictionary = [NSJSONSerialization JSONObjectWithData:data options:0 error:&parseError];
    NSLog(@"%@",responseDictionary);
    dispatch_semaphore_signal(sema);
  }
}];
[dataTask resume];
dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
```

> The above command returns JSON structured like this:

```json
{
  "plan_id": "304908b5-2843-21c9-9d79-67b765da04f0",
  "client_id": "2g8die7vki53974mp72sa4fgp5",
  "items": [
    {
      "item": {
        "license_type": "SINGLE",
        "name": "Kidstream: Monthly Access",
        "status": "ACTIVE",
        "renew_status": "RENEW",
        "renews_on": "2020-10-23T16:35:28+00:00",
        "interval_length": 1,
        "interval_unit": "MONTHS",
        "billing_cycle_anchor": "2020-10-23T09:35:28-07:00",
        "date_added": "2020-10-16T09:35:27-07:00",
        "trial": true,
        "trial_duration": 7,
        "trial_reset": 700
      },
      "license": {
        "id": "b90831a7-58bf-4ab3-9fd1-04594e635f6a",
        "internal_id": "KS001",
        "name": "Monthly Access",
        "description": "Monthly full access to Kidstream",
        "locales": [
          {
            "name": "United States",
            "country_code": "US"
          }
        ]
      }
    }
  ]
}
```
This endpoint retrieves the authenticated user's licenses to the requesting client's app.

### HTTP Request

`GET https://api.paket.tv/license`

**Authorization** OAuth 2.0

### Response Object

Attribute | Type | Description
--------- | ------- | -----------
`plan_id` | <small>string</small> | The authenticated user's plan ID
`client_id` | <small>string</small> | The requesting client's ID
`items` | <small>array</small> | An array of plan item objects

### Plan Item Object

Attribute | Type | Description
--------- | ------- | -----------
`item` | <small>object</small> | An item in the authenticated user's plan containing a license to the requesting app
`license` | <small>object</small> | The license associated with the plan item

### Item Object

Attribute | Type | Description
--------- | ------- | -----------
`license_type` | <small>string</small> | Whether the license is a la carte (`SINGLE`) or part of a bundle (`BUNDLE`)
`name` | <small>string</small> | The name of the app and associated license
`status` | <small>string</small> | The current status of the item and associated license (`ACTIVE`,`OVERDUE`, or `CANCELLED`)
`renew_status` | <small>string</small> | The renewal status of the item and associated license (`RENEW`, or `CANCEL`)
`renews_on` | <small>string</small> | The date on which the item is scheduled to renew
`interval_length` | <small>integer</small> | The interval at which the subscription renews
`interval_unit` | <small>string</small> | The unit description for the interval_length parameter
`billing_cycle_anchor` | <small>string</small> | The anchor date to which the subscription is renewed
`date_added` | <small>string</small> | The date on which the item was added to the authenticated user's plan
`trial` | <small>boolean</small> | Whether or not the item was originally added as part of a trial
`trial_duration` | <small>integer</small> | The original trial duration in days
`trial_reset` | <small>integer</small> | The trial reset period in days

### License Object

Attribute | Type | Description
--------- | ------- | -----------
`id` | <small>string</small> | The Paket assigned license ID
`internal_id` | <small>string</small> | The service provider assigned license ID
`name` | <small>string</small> | The name of the license
`description` | <small>string</small> | The license description
`locales` | <small>array</small> | Array of license locale objects containing an ISO 3166 `name` and `country_code`

# Profile

# Plan