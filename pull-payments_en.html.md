---
title: QIWI Wallet Pull REST API 2.1

search: true

metatitle: QIWI Wallet Pull REST API 2.1

metadescription: This API is to be implemented by a merchant to support Visa QIWI Wallet Pull Payments. Pull payments are those initiated from the merchant’s website, in contrast to Push payments that are initiated in Visa QIWI Wallet interfaces such as web (qiwi.com), mobile applications and self-service terminals.

services:
 - <a href='#'>Swagger</a>  |  <a href='#'>Qiwi Demo</a>

toc_footers:
 - <a href='/index-en.html'>Home page</a>
 - <a href='http://pullapi-test.qiwi.com'>Sandbox</a>

includes:
  - pull-payments/pull-payments-api_en
  - pull-payments/webform_en
  - pull-payments/checkout_en
  - pull-payments/notification_en

---

# Pull Payments {#intro}

###### Last update: 2017-10-20 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/pull-payments_en.html.md)

QIWI Pull Payments API opens a way to operations with Visa QIWI Wallet invoices from your service. The following operations are supported:

* creating invoice
* cancelling unpaid invoice
* making refund for paid invoice (a user rejects merchant's good or service)
* checking operation's status
* starting payment on QIWI Checkout

## Payment methods {#payment_methods}

* Customers may pay for Visa QIWI Wallet invoices
  * on QIWI checkout (bill.qiwi.com)
  * on [qiwi.com](#https://qiwi.com) web-site
    * from their Visa QIWI Wallet,
    * from mobile phone account or any Visa/MasterCard
  * in QIWI mobile applications (Android/iOS/Windows Phone)
    * from their Visa QIWI Wallet,
    * from mobile phone account or any Visa/MasterCard
  * by cash in QIWI Self-service kiosks.

**To use API, complete [registration and approvement of the agreement](https://ishop.qiwi.com).**

## Integration methods {#implement}

You can use the following integration methods with QIWI Wallet Pull Payments API:

* [Online Invoicing Form](#webform_en) - Quick and easy solution for invoicing (does not require user redirect to QIWI Checkout). Has limited functions - only invoice issue. Calling web form is authorized by [API ID](#auth_param) and digital signature of the request or may be used without authorization (**not recommended**).

* [Pull REST API](#pull_rest_api) - Fully functional RESTful API for all operations with invoices.
