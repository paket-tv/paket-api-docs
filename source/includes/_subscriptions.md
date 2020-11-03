# Subscriptions

The Paket Subscriptions API currently supports the following endpoints:  

- `GET /subscriptions`

## GET /subscriptions

> Get subscriptions example request 

```ruby
require "uri"
require "net/http"

url = URI("https://api.paket.tv/v1/subscriptions")

https = Net::HTTP.new(url.host, url.port);
https.use_ssl = true

request = Net::HTTP::Get.new(url)
request["Authorization"] = "eyJz9sdfsdfsdfsd"

response = https.request(request)
puts response.read_body
```

```python
import http.client
import mimetypes
conn = http.client.HTTPSConnection("api.paket.tv")
payload = ''
headers = {
  'Authorization': 'eyJz9sdfsdfsdfsd'
}
conn.request("GET", "/v1/subscriptions", payload, headers)
res = conn.getresponse()
data = res.read()
print(data.decode("utf-8"))
```

```shell
curl https://api.paket.tv/v1/subscriptions \
  -X GET \
  -H 'Authorization: eyJz9sdfsdfsdfsd'
```

```javascript
var myHeaders = new Headers();
myHeaders.append("Authorization", "eyJz9sdfsdfsdfsd");

var requestOptions = {
  method: 'GET',
  headers: myHeaders,
  redirect: 'follow'
};

fetch("https://api.paket.tv/v1/subscriptions", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));
```

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
Request request = new Request.Builder()
  .url("https://api.paket.tv/v1/subscriptions")
  .method("GET", null)
  .addHeader("Authorization", "eyJz9sdfsdfsdfsd")
  .build();
Response response = client.newCall(request).execute();
```

```swift
import Foundation

var semaphore = DispatchSemaphore (value: 0)

var request = URLRequest(url: URL(string: "https://api.paket.tv/v1/subscriptions")!,timeoutInterval: Double.infinity)
request.addValue("eyJz9sdfsdfsdfsd", forHTTPHeaderField: "Authorization")

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

NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"https://api.paket.tv/v1/subscriptions"]
  cachePolicy:NSURLRequestUseProtocolCachePolicy
  timeoutInterval:10.0];
NSDictionary *headers = @{
  @"Authorization": @"eyJz9sdfsdfsdfsd"
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

> Example response <span style="float:right">200 OK</span>

```json
{
  "plan_id": "304908b5-2843-21c9-9d79-67b765da04f0",
  "client_id": "2g8die7vki53974mp72sa4fgp5",
  "items": [
    {
      "item": {
        "license_type": "SINGLE",
        "name": "Kidstream",
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
        "rank": 0,
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

The `/subscriptions` endpoint retrieves only the authenticated user's subscriptions to the requesting client's app. The subscription items are returned in order of descending rank, with the subscription offering the highest service level in the first position.

### HTTP Request

`GET https://api.paket.tv/v1/subscriptions`

**Authorization** OAuth 2.0

Subscription types currently available include:

- `SINGLE` - The subscription license belongs to a single application item
- `BUNDLE` - The subscription license belongs to a bundle item

The `status` attribute represents the current status of the subscription, whereas the `renew_status` indicates if the subscription is slated for renewal.

The subscriptions' available statuses are:

- `ACTIVE` - The subscription is currently active and in good standing
- `OVERDUE` - The subscription is active, though there is a payment processing issue pending resolution
- `CANCELLED` - The subscription has been cancelled (returns cancelled subscriptions for past 12 months)

The subscriptions' available renew statuses are:

- `RENEW` - The subscription is scheduled to renew on the `renews_on` date
- `CANCEL` - The subscription is scheduled to be cancelled on the `renews_on` date

The Paket API returns all `ACTIVE` and `OVERDUE` subscriptions and returns the previous 12 months' `CANCELLED` subscriptions.