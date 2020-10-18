# Licenses

The Paket Licenses API currently supports the following endpoints:  

- `GET /licenses`

## GET /licenses

```ruby
require "uri"
require "net/http"

url = URI("https://api.paket.tv/licenses")

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
conn.request("GET", "/licenses", payload, headers)
res = conn.getresponse()
data = res.read()
print(data.decode("utf-8"))
```

```shell
curl --location --request GET 'https://api.paket.tv/licenses' \
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

fetch("https://api.paket.tv/licenses", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));
```

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
Request request = new Request.Builder()
  .url("https://api.paket.tv/licenses")
  .method("GET", null)
  .addHeader("Authorization", "<access_token>")
  .build();
Response response = client.newCall(request).execute();
```

```swift
import Foundation

var semaphore = DispatchSemaphore (value: 0)

var request = URLRequest(url: URL(string: "https://api.paket.tv/licenses")!,timeoutInterval: Double.infinity)
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

NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"https://api.paket.tv/licenses"]
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
> <span style="color: green">`200 OK`</span>

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
> If no licenses are available, API returns: <span style="color: green">`204 No Content`</span>

The `/licenses` endpoint retrieves only the authenticated user's licenses to the requesting client's app.

### HTTP Request

`GET https://api.paket.tv/licenses`

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
