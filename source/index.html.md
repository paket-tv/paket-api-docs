---
title: Paket API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: cURL
  - javascript: Javascript
  - ruby: Ruby
  - python: Python
  - java: Java
  - swift: Swift
  - objective_c: Objective-C

toc_footers:
  # - <a href='https:/publisher.paket.tv'>Sign In to the Publisher Portal</a>
  # - <a href='https://publisher.paket.tv/signup'>Request Access</a>

copyright: 
  - <a>© 2020-2021 Paket Media, Inc.</a>
  - <a href="https://publisher.paket.tv/privacy-policy" target="_blank">Privacy Policy</a> | <a href="https://publisher.paket.tv/terms-of-service" target="_blank">Terms of Service</a>

includes:
  - authentication
  - subscriptions
  - errors

search: false

code_clipboard: true
---

# Introduction 

Welcome to Paket. Paket is a utility platform for streaming media services that unifies authentication, billing, and discovery into a seamless and frictionless experience for users.

While nearly all of the Platform's business logic (including account creation, billing, recurring payments, etc.) happens on Paket's end, streaming service providers will need to integrate with Paket's API to be able to authenticate users and to access the authenticated user's entitlements to the respective service via the Paket API.

Also, please note that this documentation illustrates how to call the API using the standard methods employed by various software development frameworks. Framework-specific examples will be updated with community SDK examples as such SDKs are warranted and made available.

# Getting Started

> API Endpoint  

```curl
  https://api.paket.tv/v1/
```
  
Before we begin, it's important to understand the key concepts and taxonomy of the platform, each of which can be configured in the [Paket Publisher Portal](https://publisher.paket.tv):

**Companies**: The Company is the primary object to which Apps are associated. This is where the business details are configured. In most situations, an account will have only one Company;

**Apps**: The App is the primary object to which Clients and Licenses are associated. This is the high-level representation of your service, its consumer-facing title (e.g., Hulu), description, and supporting metadata;

**Licenses**: The License object represents an entitlement offered via your App (e.g., Limited Ads at $5.99 / mo, No Ads at $9.99 / mo, etc.), its pricing, and regional availability;

**Subscriptions**: The items returned when calling the `/subscriptions` api endpoint. Each item represents either a subscription to an app or bundle and contains a reference the corresponding License; and

**Clients**: The Client object contains the platform or device-specific Paket auth (OAuth2) configuration for a service's discrete user-facing application (e.g., Roku, iOS, Android, etc.).

While this documentation illustrates how to use the Paket API, service providers are encouraged to configure additional logic on their end (e.g., creating a local user from the authenticated Paket user) so as to deliver the optimal experience to the end user. 

## Statuses 

The following Statuses are assignable to the App and License models:

Status        | Description
------------- | -----------
Draft | **Default.** The object is not yet available to end users.  
Pending | Publisher has deployed the object and is pending Paket approval. The object is not available to end users. 
Live | Publisher has deployed the object and Paket has approved it. The object is available to end users.  
Inactive | Publisher has hidden a Paket-approved object that has active subscribers. The object is not available to new end users.

## Price Tiers

Paket uses price tiers to ensure pricing consistency and predictability across regions and countries. Pricing and regional availability are assigned to the `License` object. You may create as many licenses as is necessary to ensure that your regional pricing is accurately represented.

<aside class="notice">
  Price tier changes on Live licenses can be scheduled via request to Paket and require a min 30 days lead time before it will be applied to existing subscribers.
</aside>

## Testing

Testing on Paket is done in the Production environment using test mode Client and License objects. This ensures that you are testing your API integration in the very environment to which you will be deploying your App. 

Simply create one or more test mode Client and License objects within your App by selecting the "Test Mode" switch within the Publisher Portal. This will allow you to test your implementation using the test client.  

**A test mode Client will retrieve all of the App's test mode licenses that are set to status Pending.**

When you are ready to deploy your app, replace the test mode client parameters with those of the production client.