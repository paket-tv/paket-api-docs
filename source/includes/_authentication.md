# Authentication

The Paket API employs user access tokens to allow access to the API. Access tokens are issued to the requesting client on successful user authentication using the Paket OAuth2 authorization code grant flow.  

In nearly all cases, this flow will work with your existing web, mobile, and input-restricted client flows. If needed, Paket offers its own code-based flow for input-restricted devices. If you'd like to learn more, please contact us.  

While not officially tested, the authorization flow should work with most framework-specific Oauth2 clients - including those with full PKCE support.

<aside class="notice">
  For authorizing users in native apps and in keeping with OAuth2 <a href='https://www.rfc-editor.org/rfc/rfc8252.txt' target='_blank'>best practices</a>, please perform the authorization request in an external user agent (i.e., browser) rather than an embedded user agent.
</aside>

## Integrating

Before authenticating with Paket, you’ll need to configure a new Client under your App settings via the publisher console. Once configured, note the following authentication server endpoints:

- **AUTHORIZE** Endpoint: `https://auth.paket.tv/oauth2/authorize`  
- **TOKEN** Endpoint: `https://auth.paket.tv/oauth2/token`  
- **LOGOUT** Endpoint: `https://auth.paket.tv/logout`  

To integrate your service with Paket, use the following flow:

> A basic example of the constructed URL:  

```
GET https://auth.paket.tv/oauth2/authorize?response_type=code
&client_id=2g8die7vki53974mp72sa4fgp5&redirect_uri=www.example.com/oauth/callback
```

### 1. Redirect users to request Paket access

`GET https://auth.paket.tv/oauth2/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI`

When redirecting a user to Paket to authorize access to your application, you’ll need to construct the authorization URL with the following parameters:

Parameter | Type | Description
--------- | ------- | -----------
`response_type` | string | **Required** Value `code`
`client_id` | string | **Required** The requesting client's ID as specified in the publisher portal
`redirect_uri` | string | **Required** The client Callback URL as specified in the publisher portal
`state` | string | **Optional** An unguessable random string, used to protect against cross-site request forgery attacks
`code_challenge_method` | string | **Optional** When using PKCE (recommended) Paket supports only `S256`
`code_challenge` | string  | **Optional** Required only when the `code_challenge_method` is specified  

### 2. Paket redirects back to your site  

Once the user logs in, Paket will redirect them back to your `redirect_uri` with a temporary `code` parameter. If you specified a  
`state` parameter in Step 1, it will be returned as well. The `state` parameters should always match. If they do not, the request should not be trusted.

Example of the redirect back to your URL:

`GET https://www.example.com/oauth/callback?code=b33b3e29-9a68-46ab-ae37-f2a05bf9d4b6`  

### 3. Exchange code for an access token

```shell
curl https://auth.paket.tv/oauth2/token \
  -X POST \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=authorization_code&code=4b33b3e29-9a68-46ab-ae37-f2a05bf9d4b6
  &client_id=2g8die7vki53974mp72sa4fgp5&redirect_uri=https://example.com/oauth/callback'
```

```javascript
var urlencoded = new URLSearchParams();
urlencoded.append("grant_type", "authorization_code");
urlencoded.append("code", "4b33b3e29-9a68-46ab-ae37-f2a05bf9d4b6");
urlencoded.append("client_id", "2g8die7vki53974mp72sa4fgp5");
urlencoded.append("redirect_uri", "https://example.com/oauth/callback");

var requestOptions = {
  method: 'POST',
  body: urlencoded,
  redirect: 'follow'
};

fetch("https://auth.paket.tv/oauth2/token", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));
```

```ruby
require "uri"
require "net/http"

url = URI("https://auth.paket.tv/oauth2/token")

https = Net::HTTP.new(url.host, url.port);
https.use_ssl = true

request = Net::HTTP::Post.new(url)
request.body = "grant_type=authorization_code&code=4b33b3e29-9a68-46ab-ae37-f2a05bf9d4b6&client_id=2g8die7vki53974mp72sa4fgp5&redirect_uri=https%3A//example.com/oauth/callback"

response = https.request(request)
puts response.read_body
```

```python
import http.client
import mimetypes
conn = http.client.HTTPSConnection("auth.paket.tv")
payload = 'grant_type=authorization_code&code=4b33b3e29-9a68-46ab-ae37-f2a05bf9d4b6&client_id=2g8die7vki53974mp72sa4fgp5&redirect_uri=https%3A//example.com/oauth/callback'
headers = {}
conn.request("POST", "/oauth2/token", payload, headers)
res = conn.getresponse()
data = res.read()
print(data.decode("utf-8"))
```

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("text/plain");
RequestBody body = RequestBody.create(mediaType, "grant_type=authorization_code&code=4b33b3e29-9a68-46ab-ae37-f2a05bf9d4b6&client_id=2g8die7vki53974mp72sa4fgp5&redirect_uri=https://example.com/oauth/callback");
Request request = new Request.Builder()
  .url("https://auth.paket.tv/oauth2/token")
  .method("POST", body)
  .build();
Response response = client.newCall(request).execute();
```

```swift
import Foundation

var semaphore = DispatchSemaphore (value: 0)

let parameters = "grant_type=authorization_code&code=4b33b3e29-9a68-46ab-ae37-f2a05bf9d4b6&client_id=2g8die7vki53974mp72sa4fgp5&redirect_uri=https%3A//example.com/oauth/callback"
let postData =  parameters.data(using: .utf8)

var request = URLRequest(url: URL(string: "https://auth.paket.tv/oauth2/token")!,timeoutInterval: Double.infinity)
request.httpMethod = "POST"
request.httpBody = postData

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

NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"https://auth.paket.tv/oauth2/token"]
  cachePolicy:NSURLRequestUseProtocolCachePolicy
  timeoutInterval:10.0];
NSMutableData *postData = [[NSMutableData alloc] initWithData:[@"grant_type=authorization_code" dataUsingEncoding:NSUTF8StringEncoding]];
[postData appendData:[@"&code=4b33b3e29-9a68-46ab-ae37-f2a05bf9d4b6" dataUsingEncoding:NSUTF8StringEncoding]];
[postData appendData:[@"&client_id=2g8die7vki53974mp72sa4fgp5" dataUsingEncoding:NSUTF8StringEncoding]];
[postData appendData:[@"&redirect_uri=https://example.com/oauth/callback" dataUsingEncoding:NSUTF8StringEncoding]];
[request setHTTPBody:postData];

[request setHTTPMethod:@"POST"];

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


> After a successful request, a valid access token will be returned in the response:  <span style="float:right">200 OK</span>

```json
{ 
  "access_token": "eyJz9sdfsdfsdfsd", 
  "refresh_token": "dn43ud8uj32nk2je", 
  "id_token": "dmcxd329ujdmkemkd349r",
  "token_type": "Bearer", 
  "expires_in": 3600
}
```

After you have received the temporary `code`, you can exchange it for valid access and refresh tokens. This can be done by making a POST call to the TOKEN Endpoint:

`POST https://auth.paket.tv/oauth2/token`  

With the following parameters:  

Parameter | Type | Description
--------- | ------- | ------ 
`grant_type` | string | **Required** Value `authorization_code`
`code` | string | **Required** The `code` query parameter from Step 2
`client_id` | string | **Required** The requesting client's ID as specified in the publisher portal
`redirect_uri` | string | **Required** Must be the same `redirect_uri` that was used in Step 1
`code_verifier` | string | **Optional** Required only if the authorization code was requested with PKCE.

And the following request headers:

Parameter | Type | Value
--------- | ------- | ------
`Content-Type` | string | `application/x-www-form-urlencoded` 

### 4. Make an API call

After you have a valid access token, you can make your first API call by including the `access_token` as the `Authorization` header value.

## Refresh Tokens

```shell
curl https://auth.paket.tv/oauth2/token \
  -X POST \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=refresh_token&refresh_token=dn43ud8uj32nk2je
  &client_id=2g8die7vki53974mp72sa4fgp5'
```

```javascript
var urlencoded = new URLSearchParams();
urlencoded.append("grant_type", "refresh_token");
urlencoded.append("refresh_token", "dn43ud8uj32nk2je");
urlencoded.append("client_id", "2g8die7vki53974mp72sa4fgp5");

var requestOptions = {
  method: 'POST',
  body: urlencoded,
  redirect: 'follow'
};

fetch("https://auth.paket.tv/oauth2/token", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));
```

```ruby
require "uri"
require "net/http"

url = URI("https://auth.paket.tv/oauth2/token")

https = Net::HTTP.new(url.host, url.port);
https.use_ssl = true

request = Net::HTTP::Post.new(url)
request.body = "grant_type=refresh_token&refresh_token=dn43ud8uj32nk2je&client_id=2g8die7vki53974mp72sa4fgp5"

response = https.request(request)
puts response.read_body
```

```python
import http.client
import mimetypes
conn = http.client.HTTPSConnection("auth.paket.tv")
payload = 'grant_type=refresh_token&refresh_token=dn43ud8uj32nk2je&client_id=2g8die7vki53974mp72sa4fgp5'
headers = {}
conn.request("POST", "/oauth2/token", payload, headers)
res = conn.getresponse()
data = res.read()
print(data.decode("utf-8"))
```

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("text/plain");
RequestBody body = RequestBody.create(mediaType, "grant_type=refresh_token&refresh_token=dn43ud8uj32nk2je&client_id=2g8die7vki53974mp72sa4fgp5");
Request request = new Request.Builder()
  .url("https://auth.paket.tv/oauth2/token")
  .method("POST", body)
  .build();
Response response = client.newCall(request).execute();
```

```swift
import Foundation

var semaphore = DispatchSemaphore (value: 0)

let parameters = "grant_type=refresh_token&refresh_token=dn43ud8uj32nk2je&client_id=2g8die7vki53974mp72sa4fgp5"
let postData =  parameters.data(using: .utf8)

var request = URLRequest(url: URL(string: "https://auth.paket.tv/oauth2/token")!,timeoutInterval: Double.infinity)
request.httpMethod = "POST"
request.httpBody = postData

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

NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"https://auth.paket.tv/oauth2/token"]
  cachePolicy:NSURLRequestUseProtocolCachePolicy
  timeoutInterval:10.0];
NSMutableData *postData = [[NSMutableData alloc] initWithData:[@"grant_type=refresh_token" dataUsingEncoding:NSUTF8StringEncoding]];
[postData appendData:[@"&refresh_token=dn43ud8uj32nk2je" dataUsingEncoding:NSUTF8StringEncoding]];
[postData appendData:[@"&client_id=2g8die7vki53974mp72sa4fgp5" dataUsingEncoding:NSUTF8StringEncoding]];
[request setHTTPBody:postData];

[request setHTTPMethod:@"POST"];

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

> After a successful request, a valid access token will be returned in the response:  <span style="float:right">200 OK</span>

```json
{ 
  "access_token": "eyJz9sdfsdfsdfsd", 
  "id_token": "dmcxd329ujdmkemkd349r",
  "token_type": "Bearer", 
  "expires_in": 3600
}
```

A refresh token is issued only once when you first authenticate using the `authorization_code` flow. Since the access token expires after one hour, you can request a new access token by passing the `refresh_token` to the TOKEN Endpoint and using the  
`refresh_token` grant type. While the default expiration of the refresh token is thirty (30) days, you can modify this parameter via the publisher console.

To get a new access token, send a POST request to the TOKEN Endpoint just like before:

`POST https://auth.paket.tv/oauth2/token`

Only this time, with the following parameters:

Parameter | Type | Description
--------- | ------- | ------ 
`grant_type` | string | **Required** Value `refresh_token`
`refresh_token` | string | **Required** The refresh token received in Step 3
`client_id` | string | **Required** The requesting client's ID as specified in the publisher portal

And the following request headers:

Parameter | Type | Value
--------- | ------- | ------
`Content-Type` | string | `application/x-www-form-urlencoded` 

## Revoking Access

> An example of the constructed logout URL:  

```
GET https://auth.paket.tv/logout?client_id=2g8die7vki53974mp72sa4fgp5
```

`GET https://auth.paket.tv/logout?client_id=CLIENT_ID`

To log a user out of your client and disconnect your application’s access to the user’s account, redirect the user to the LOGOUT Endpoint with the following parameters:  

Parameter | Type | Description
--------- | ------- | ------ 
`client_id` | string | **Required** The requesting client's ID as specified in the publisher portal
`logout_uri` | string | **Optional** The `logout_uri` specified in your client configuration