---
title: Paket API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby
  - python
  - javascript

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

> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Licenses

## Get Licenses

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

# Profile

# Plan