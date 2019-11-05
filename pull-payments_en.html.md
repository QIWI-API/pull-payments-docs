---
title: QIWI Wallet Pull REST API 2.1

search: true

metatitle: QIWI Wallet Pull REST API 2.1

metadescription: This API provides access to operations with invoices in QIWI Wallet service. Invoice is the unique request for the payment. The user may pay the invoice with any accessible means till the invoice expired. API supports creating invoices, cancelling unpaid invoices, making refunds for paid invoices (when a user rejects merchant's good or service), checking operation's status.

language_tabs:
  - http
  - php

services:
 - <a href='#'>Swagger</a>  |  <a href='#'>Qiwi Demo</a>

toc_footers:
 - <a href='/en/'>Home page</a>

includes:
  - pull-payments/pull-payments-api_en
  - pull-payments/webform_en
  - pull-payments/checkout_en
  - pull-payments/notification_en

---

# Pull Payments {#intro}

###### Last update: 2017-11-15 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/pull-payments_en.html.md)

QIWI Pull Payments API opens a way to operations with QIWI Wallet invoices from your service. The following operations are supported:

* creating invoice
* cancelling unpaid invoice
* making refund for paid invoice (a user rejects merchant's good or service)
* checking operation's status
* starting payment on QIWI Checkout

## Payment methods {#payment_methods}

* Customers may pay for QIWI Wallet invoices
  * on QIWI checkout (oplata.qiwi.com)
  * on [qiwi.com](#https://qiwi.com) web-site
    * from their QIWI Wallet,
    * from mobile phone account or any Visa/MasterCard
  * in QIWI mobile applications (Android/iOS/Windows Phone)
    * from their QIWI Wallet,
    * from mobile phone account or any Visa/MasterCard
  * by cash in QIWI Self-service kiosks.

**To use API, complete [registration and approvement of the agreement](https://kassa.qiwi.com).**

## Integration methods {#implement}

You can use the following integration methods with QIWI Wallet Pull Payments API:

* [Online Invoicing Form](#webform_en) - Quick and easy solution for invoicing (does not require user redirect to QIWI Checkout). Has limited functions - only invoice issue. You can call the web form in two ways:
    * authorized by [API ID](#auth_param) and digital signature of the request
    * without authorization (**not recommended**)

* [Pull REST API](#pull_rest_api) - Fully functional RESTful API for all operations with invoices.
