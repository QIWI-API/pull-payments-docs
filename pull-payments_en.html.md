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
  - pull-payments/responses_en
  - pull-payments/statuses_en
  - pull-payments/errors_en

---

# Introduction {#intro}

###### Last update: 2017-05-31 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/pull-payments_en.html.md)

This API is to be implemented by a merchant to support Visa QIWI Wallet Pull Payments. Pull payments are those initiated from the merchant’s website, in contrast to Push payments that are initiated in Visa QIWI Wallet interfaces such as web (qiwi.com), mobile applications and self-service terminals.

The following operations are supported: 

* creating invoice, 
* cancelling invoice, 
* making refund for paid invoice, 
* checking invoice payment status and refund payment status.

## Payment methods {#payment_methods}

* Customers may pay for pull payments by various methods
  * on QIWI checkout (bill.qiwi.com)
  * on [qiwi.com](#https://qiwi.com) site
    * from their Visa QIWI Wallet, 
    * from mobile phone account, 
    * from any Visa/MasterCard card
  * in QIWI Wallet mobile applications (Android/iOS/Windows Phone)
    * from their Visa QIWI Wallet, 
    * from mobile phone account, 
    * from any Visa/MasterCard
  * by cash in self-service terminals. 

**API is accessible only after the [registration and approvement of the request](https://ishop.qiwi.com).**

## Integration methods

You can use one of the following integration methods on QIWI Wallet Pull Payments API:

* [Online Invoicing Form](#webform_en) - a web invoicing form on QIWI side. Quick and easy solution for invoicing with limited functions (only invoice issue). Web-form address: 

`https://bill.qiwi.com/order/external/create.action`

* [Pull REST API](#pull_rest_api) - fully functional REST-based API for managing invoices and refunding paid invoices.

## Service data {#auth_param}

<ul class="nestedList params">
    <li><h3>Authorization and web form</h3><span>Data can be obtained on <a href="https://ishop.qiwi.com">ishop.qiwi.com</a></span>
    </li>
</ul>

`Parameter|Description|Type|Required
 ---------|--------|---|------
 API_ID | Merchant’s shop unique identifier for Visa QIWI Wallet API | Integer| +
 API_PASSWORD | Password corresponding to the API_ID. Each API_ID has a different password.| String | +
 Shop ID | Merchant's numeric Shop ID | Integer | +



<aside class="notice">
To obtain service data, visit <i>Protocols details</i> - <i>REST-protocol</i> section of <a href='http://ishop.qiwi.com'>ishop.qiwi.com</a>.
</aside>


