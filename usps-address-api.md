# USPS Addresses 3.0 API

<https://developers.usps.com/getting-started>

We go to the following address to make an account and to get an API token:

<https://cop.usps.com/cop-navigator>

We navigate to "My Apps", and then to "Developer Apps". We make an app.

> **App FAQs**
> 
> **What is an App?**
>
> Apps are API consumers registered with USPS APIs. Apps are registered to obtain credentials that enable access to one or more API products (or a set URIs).
>
> **What is an API Product?**
>
> An API product consists of a bundle of API resources (URIs) provided by USPS APIs.
>
> **Some examples;**
>
> "Public Access" API Product (addresses-v3, domestic-prices-v3, international-prices-v3, locations-v3, oauth2-userinfo, oauth2-v3, service-standards-files-v3, service-standards-v3, shipments-v3)
"Shipping Suite" API Product (All endpoints from Public Access including adjustments-v3, appointments-v3, carrier-pickup-v3, containers-v3, disputes-v3, international-labels-v3, labels-v3, payments-v3, pmod-v3, scan-form-v3, tracking-v3)
>
> **What is an API Key?**
>
> An API key (also known as a consumer key) is a string value passed by a client app to USPS API proxies to authenticate this client app. The key uniquely identifies your client app.

We make credentials. There is a Consumer Key (=Client ID in Postman) and a Consumer Secret (=Client Secret in Postman).

We download the OAuth 2.0 OpenAPI specification at <https://developers.usps.com/Oauth>.

We use Postman and get a temporary OAuth token.

We download the Addresses 3.0 OpenAPI specification at <https://developers.usps.com/addressesv3>

We use the OAuth token to use this API.


