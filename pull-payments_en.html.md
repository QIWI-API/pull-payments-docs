---
title: QIWI Wallet Pull REST API 2.1

search: true

metatitle: QIWI Wallet Pull REST API 2.1

metadescription: This API is to be implemented by a merchant to support Visa QIWI Wallet Pull Payments. Pull payments are those initiated from the merchant’s website, in contrast to Push payments that are initiated in Visa QIWI Wallet interfaces such as web (qiwi.com), mobile applications and self-service terminals.

language_tabs:
  - shell
  - php: PHP
  - http: HTTP
  - xml: XML 
  - json: JSON

services:
 - <a href='#'>Swagger</a>  |  <a href='#'>Qiwi Demo</a>

toc_footers:
 - <a href='/'>Home page</a>
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

###### Last update: 2017-05-31 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/pullrest_en.html.md)

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

## Integrations

## Request Content-type  

The following `Content-type` is accepted:

`application/x-www-form-urlencoded; charset=utf-8`  

## Response Content-type

Content-type of the response has to be specified in the request's "Accept" header. The following responses are supported:

`application/json`

`text/json`

`text/xml`

`application/xml`

## Authorization {#auth}

Type | Description
---|---
basic | Merchant service is authorized by API ID and API password generated for the merchant's account in https://ishop.qiwi.com.<ul><li>API ID – merchant’s shop unique identifier for Visa QIWI Wallet API, as displayed in API ID parameter of *REST-protocol* section;</li><li>API password – Visa QIWI Wallet API password corresponding to this API ID.</li></ul>

# Invoicing Operation Flow

<img src="images/pullrest_1_en.png" />

1. User submits an order on the merchant’s website. 
2. Merchant sends [Create invoice](#invoice) request to Visa QIWI Wallet server with authorization parameters.
3. Merchant is recommended to redirect to [QIWI Checkout](#checkout) page on Visa QIWI Wallet site when the request is completed. Otherwise, invoice can be paid in any Visa QIWI Wallet interfaces, such as web (qiwi.com), mobile applications and self-service terminals.
4. If merchant enables [notifications](#notification), then Visa QIWI Wallet sends to the merchant's server a notification on the invoice status once invoice is paid or cancelled by the user. Authorization on the merchant's side is required for notifications.
5. Merchant can [request current status](#invoice-status) of the created invoice, or [cancel invoice](#cancel) (provided that it has not been paid yet) at any moment.
6. Merchant delivers ordered services/goods when the invoice gets paid.

<!--# Integration Methods

You can use one of the following integration methods on QIWI Pull Payments:

* Pull REST API
* Web Payment Form
-->

# Issuing Invoice for the Order {#invoice}

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/test234578"
  -X PUT 
  -d 'user=tel%3A%2B79161111111&amount=1.00&ccy=RUB&comment=uud_TEST7&lifetime=2016-09-25T15:00:00'
  --header "Accept: text/json" --header "Authorization: Basic ***"  
~~~

~~~http
PUT /api/v2/prv/2042/bills/BILL-1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

user=tel%3A%2B79031234567%26amount=10.0%26ccy=RUB%26comment=test%26lifetime=2012-11-25T09%3A00%3A00
~~~

Request creates new invoice to the specified phone number which conincides with wallet ID in QIWI Wallet. Request type - HTTP PUT. 

Parameters in the PUT-request's pathname:

* `{prv_id}` - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of ishop.qiwi.com web site);
* `{bill_id}` - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters).

Request URL:

`https://api.qiwi.com/api/v2/prv/{prv_id}/bills/{bill_id}`

Parameters are sent in the request body as formdata.

Parameter|Description|Type|Required
---------|--------|---|------
user | The Visa QIWI Wallet user’s ID, to whom the invoice is issued. It is the user’s phone number with "tel:" prefix | String(20)|Y
amount | The invoice amount. The rounding up method depends on the invoice currency | Number(6.3)|Y
ccy | Invoice currency identifier (Alpha-3 ISO 4217 code). Depends on currencies allowed for the merchant. The following values are supported: RUB, EUR, USD, KZT | String(3)|Y
comment | Comment to the invoice | String(255)|Y
lifetime | Date and time up to which the invoice is available for payment. If the invoice is not paid by this date it will become void and will be assigned a final status.<br> **Important! Invoice will be automatically expired when 45 days is passed after the invoicing date**|dateTime|Y
pay_source |If the value is "mobile" the user’s MNO balance will be used as a funding source. If the value is "qw", any other funding source is used available in Visa QIWI Wallet interface. If parameter isn’t present, value "qw" is assumed |String |N 
prv_name|Merchant’s name| String(100)|N

[Response parameters](#response_bill)

[Error response](#response_error)

# Requesting Invoice Status {#invoice-status}

Merchant can request payment status of the invoice by sending the following GET-request.

~~~http
GET /api/v2/prv/2042/bills/BILL-1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8
~~~

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/sdf23452435"
  --header "Authorization: Basic ***" --header "Accept: text/json" 
~~~

Parameters in the request's pathname:

* `{prv_id}` - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of ishop.qiwi.com web site);
* `{bill_id}` - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters).

Request URL:

`https://api.qiwi.com/api/v2/prv/{prv_id}/bills/{bill_id}`

[Response parameters](#response_bill)

[Error response](#response_error)

# Cancelling Unpaid Invoice {#cancel}

Request cancels unpaid invoice.

~~~http
PATCH /api/v2/prv/2042/bills/BILL-1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

status=rejected
~~~

~~~shell
user@server:~$ curl -X PATCH 
  --header "Authorization: Basic ***" 
  --header "Accept: text/json" 
  "https://api.qiwi.com/api/v2/prv/373712/bills/sdf23452435" 
  -d 'status=rejected'
~~~

Parameters in the PATCH-request's pathname:

* `{prv_id}` - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of ishop.qiwi.com web site);
* `{bill_id}` - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters).

Required parameter in the request's body:

* `status` - "rejected" string (cancel status)

Request URL:

`https://api.qiwi.com/api/v2/prv/{prv_id}/bills/{bill_id}`

[Response parameters](#response_bill)

[Error response](#response_error)

# Refunds

Request processes a full or partial refund to user's Visa QIWI Wallet account, so a reversed transaction with the same currency is created for the initial one.

Merchant can create several refund operations for the same initial invoice provided that:

* Amount of all refund operations does not exceed initial invoice amount.
* Different refund IDs used for different refund operations of the same invoice (see below).

<aside class="warning">
When the transmitted amount exceeds the initial invoice amount or the amount left after the previous refunds, server returns error code 242.
</aside>

## Refund Operation Flow

<img src="images/pullrest_2_en.png" />

1. To refund a part of the invoice amount or the full amount, merchant sends a request for refund to Visa QIWI Wallet server.
2. To make sure that the payment refund has been successfully processed, merchant can periodically request the invoice [refund status](#refund_status) until the final status is received.
3. This scenario can be repeated multiple times until the invoice is completely refunded (whole invoice amount has been returned to the user).

## Request Details

Parameters in the PUT-request's pathname:

* `{prv_id}` - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of ishop.qiwi.com web site);
* `{bill_id}` - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters).
* `{refund_id}` - refund identifier, a number specific to a series of refunds for the invoice `{bill_id}` (string of 1 to 9 symbols – any 0-9 digits and upper/lower Latin letters).

Request URL:

`https://api.qiwi.com/api/v2/prv/{prv_id}/bills/{bill_id}/refund/{refund_id}`

Parameter in the request's body:

~~~shell
user@server:~$ curl -v -w "%{http_code}" -X PUT 
  --header "Accept: text/json" 
  --header "Authorization: Basic ***" 
  "https://api.qiwi.com/api/v2/prv/373712/bills/test234578/refund/122swbill" 
  -d 'amount=10.0'
~~~

~~~http
PUT /api/v2/prv/2042/bills/BILL-1/refund/122swbill HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

amount=10.0
~~~

Parameter|Description|Type|Required
---------|--------|---|------
amount | The refund amount should be less or equal to the amount of the initial transaction specified in<br> `{bill_id}`. The rounding up method depends on the invoice currency | Number(6.3)|Y

[Response parameters](#response_refund)

[Error response](#response_error)

# Check Refund Status

Merchant can verify current status of the refund by sending Refund Status GET-request.

Parameters in the GET-request's pathname:

* `{prv_id}` - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of ishop.qiwi.com web site);
* `{bill_id}` - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters).
* `{refund_id}` - refund identifier, a number specific to a series of refunds for the invoice `{bill_id}` (string of 1 to 9 symbols – any 0-9 digits and upper/lower Latin letters).

Request URL:

`https://api.qiwi.com/api/v2/prv/{prv_id}/bills/{bill_id}/refund/{refund_id}`

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/test234578/refund/122swbill"
  -v -w "%{http_code}" 
  --header "Accept: text/json" --header "Authorization: Basic ***" 
~~~

~~~http
GET /api/v2/prv/2042/bills/BILL-1/refund/122swbill HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8
~~~

[Response parameters](#response_refund)

[Error response](#response_error)

# Checkout

Merchant may offer a Visa QIWI Wallet user to pay the invoice immediately by redirecting to the Visa QIWI Wallet Сheckout page. To redirect, HTTP GET-request is used to the following URL:

`https://bill.qiwi.com/order/external/main.action`

Server opens invoice checkout page. Page contains a number of ways to pay the invoice.

## GET Parameters

Parameter|Type|Description|Required
---------|--------|---|------
shop| string |Merchant’s ID in Visa QIWI Wallet system, corresponds to `{prv\_id}` parameter used to create the bill.|Y
transaction| string |Invoice ID generated by the merchant, corresponds to `{bill\_id}` parameter used to create the bill.|Y
iframe| boolean | This parameter (if `true`) means that invoice page would be opened in "iframe". The checkout page appears more compact and can be embedded conveniently within the merchant’s site. Default value is `false`|N
successUrl |URL-encoded string| The URL to which the payer will be redirected in case of successful creation of Visa QIWI Wallet transaction. URL must be within merchant's site | N 
failUrl |URL-encoded string | The URL to which the payer will be redirected when creation of Visa QIWI Wallet transaction is unsuccessful. URL must be within merchant's site |N
target |string|"iframe" or empty. This parameter means that hyperlink specified in `successUrl` / `failUrl` parameter opens in "iframe" page|N
pay_source |string| Accepts `mobile`, `qw`, `card`, `wm`, `ssk`. Default payment method to show first for the client. Allowed values:<br> `qw` – Visa QIWI Wallet account;<br> `mobile` – client’s cell phone account;<br> `card` – a credit/debit card;<br> `wm` – linked WebMoney wallet;<br> `ssk` – payment by cash in a QIWI Terminal.<br> When specified method is inaccessible for the Customer, the page contains notice about it and the client can choose another method.|N

## Redirection to Merchan't Site

Checkout page may redirect Customer back to URL specified in `successUrl` or `failUrl` parameter (if any) when the Customer completes payment process.

<aside class="notice">
Redirection occurs when the user pays by Visa QIWI Wallet account balance only.
</aside>

<aside class="warning">
Redirection to <i>successUrl</i> does not necessarily mean that invoice is successfully paid. You should wait for the Visa QIWI Wallet’s notification with final invoice status to the merchant’s server before complete the user order. When notifications are not used, you should request the invoice status.
</aside>

The URL for redirection supplements `order` parameter with its value as the invoice ID taken from the `transaction` parameter of the original Checkout GET-request. Using this parameter, merchant can render the final page depending on the order details.

## Example

1. Merchant redirects to URL: `https://qiwi.com/order/external/main.action?shop=2042&transaction=123123123&successUrl= http%3A%2F%2Fmystore.com%2Fsuccess%3Fa%3D1%26b%3D2&failUrl= http%3A%2F%2Fmystore.com%2Ffail%3Fa%3D1%26b%3D2&iframe=true&target=iframe& pay_source=qw`

2. User pays the invoice by Visa QIWI Wallet balance on Checkout page and completes payment process.

3. If transaction is successfully created, Checkout page redirects Customer to `http://mystore.com/success?a=1&b=2&order=123123123`.

4. If creation of transaction is unsuccessful, Checkout page redirects Customer to `http://mystore.com/fail?a=1&b=2&order=123123123`.

# Notifications {#notification}

Notifications are POST-requests (callbacks) from Visa QIWI Wallet server containing all relevant data of the invoice serialized as HTTP-request parameters (encoded by UTF-8) plus parameter `command=bill`.

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
Authorization: Basic ***

bill_id=BILL-1&status=paid&error=0&amount=1.00&user=tel%3A%2B79031811737&prv_name=Retail_Store&ccy=RUB&comment=test&command=bill
~~~

To receive notifications (callbacks on invoice status changing), use **Turn on notifications** flag in **Settings - Protocols details - REST-protocol** section of [QIWI](ishop.qiwi.com) web site. To receive these notifications merchant’s server should accept specific HTTP-requests on ports 80, 443.

The notification data are transmitted as `application/x-www-form-urlencoded` content type encoded by UTF-8. Parameters, transmitted in the URL string, are URL-encoded. 

Response must be in XML format.

To receive notifications merchant must whitelist following IP subnets connected by 80, 443 ports exclusively: 

* 91.232.230.0/23
* 79.142.16.0/20

## Authorization on Merchant's Server

Merchant's server can use [basic-authorization](#basic_notify) or [authorization by signature](#sign_notify). To use HTTPS protocol, merchant's server should support SSL-encryption and client SSL certificate verification.

<aside class="notice">
If the client SSL-certificate is self-generated and is not issued by one of the standard certification centers, this certificate should be uploaded to the Visa QIWI Wallet server via the merchant’s console (<b>Certificate</b> field in <b>Settings - Protocols details - REST-protocol</b> section of <a href="https://ishop.qiwi.com">QIWI</a> web site). Certificate must be in one of the following formats:
<ul><li>PEM (text file with .pem extension) – (Privacy-enhanced Electronic Mail) BASE64 encoded DER certificate placed between `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----` strings.</li>
<li>DER (binary file with .cer, .crt, .der extensions) – usually in binary DER format, though PEM certificates are also accepted with this extensions.</li></ul>

The merchant's certificate becomes trusted after the upload.
</aside>

### Basic-authorization {#basic_notify}

Plain text password basic-authorization. It is recommended to use HTTPS protocol in this case to make request well protected.
The `Shop ID:Notification password` couple is used where:

* `Shop ID` – merchant's Shop ID, as displayed in **Shop ID** parameter of **Protocols details - REST-protocol** section of [QIWI](https://ishop.qiwi.com) web site.
* `Notification password` – password for notifications issued on merchant's registration. Merchant should change it immediately in **Protocols details - REST-protocol** section of [QIWI](https://ishop.qiwi.com) web site (**Change password** button). If necessary, merchant may reset password for notifications at any time.

### Authorization by signature {#sign_notify}

Merchant should enable **Sign** flag in **Protocols details - REST-protocol** section of [QIWI](https://ishop.qiwi.com) web site.
The HTTP header `X-Api-Signature` is added to the POST-request. Signature is calculated as HMAC algorithm with SHA1-hash function.

* Parameters are in alphabetical order and byte-encoded.
* Separator is `|`.
* Signed are all the parameters in the [request](#invoice).
* Secret key for signature is [Notification password](#basic_notify) from basic-authorization.

Signature verification algorithm:

1. Prepare a string of all parameters values from the POST-request sorted in alphabetical order and separated by `|`: `{parameter1}|{parameter2}|…` where `{parameter1}`, `{parameter2}` are notification parameters. All parameters should be treated as strings.

2. Transform obtained string and `Notification password` (see [basic-authorization](#basic_notify)) into bytes encoded in UTF-8.

3. Apply HMAC-SHA1 function:

`hash = HMAС(SHA1, Notification_password_bytes, Invoice_parameters_bytes)`

Where:

* `Notification_password_bytes` – secret key in bytecodes (Notification password);
* `Invoice_parameters_bytes` – POST-request in bytecodes;
* `hash` – hash-function result.
4. Transform into bytes UTF-8 and Base64-encode Hash HMAC value. Then compare with `X-Api-Signature` header's value.

[Notification handler example](#php_apisign)

## Requirements to the Response for Notification

The response should be in XML format.

~~~http
HTTP/1.1 200 OK
Content-Type: text/xml

<?xml version="1.0"?> 
<result>
<result_code>0</result_code>
</result>
~~~

`Content-type` HTTP header must be `text/xml` otherwise notification is treated as unsuccessful on Visa QIWI Wallet side.

In response to the request, the result code must be returned in `<result_code>` tag within `<result>` tag.

In order to help in identifying the reasons of notification errors we recommend that the result codes returned by the merchant be in accordance with [Notification codes table](#notify_codes).

Any response with result code other than 0 ("Success") and/or HTTP status code other than 200 (OK) will be treated by Visa QIWI Wallet server as a temporary error. Thus the server will continue repeating requests (notifications) with increased time intervals within next 24 hours (50 attempts in total) until it gets a response with result code 0 ("Success") and HTTP status code 200 (OK).

If the response with result code 0 ("Success") and HTTP status code 200 has not been received within 24 hours, Visa QIWI Wallet server will stop sending the requests and will send an email to the merchant with new Invoice status code and indication on the possible technical issues on the merchant’s server side.

## Notification Codes {#notify_codes}

Code|Description
--------------|--------
0|Success
5|The format of the request parameters is incorrect
13|Database connection error
150|Incorrect password
151|Signature authorization failed
300|Server connection error

## Signed Notification Authorization in PHP {#php_apisign}

Open *PHP* tab on the right.

~~~php
<?php

function hexToStr($hex){
    $string='';
    for ($i=0; $i < strlen($hex)-1; $i+=2){
        $string .= chr(hexdec($hex[$i].$hex[$i+1]));
    }
    return $string;
}

//Signature generation by key and string
function checkSign($key, $req){
    $sign_hash = hash_hmac("sha1", $req, $key);
    $sign_tr = hexToStr($sign_hash);
    $sign = base64_encode($sign_tr);
    return $sign;
}

//Sort POST-request parameters and return values
function getReqParams(){
    $reqparams = "";
    ksort($_POST);
    foreach ($_POST as $param => $valuep) {
        $reqparams = "$reqparams|$valuep";
    }
    return substr($reqparams,1);
}

//Take signature from the request
function getSign(){
    $HEADERS = getallheaders();
    foreach ($HEADERS as $header => $value) {
       if ($header == 'X-Api-Signature') {
            $SIGN_REQ = $value;
       }
    }
    return $SIGN_REQ;
}

// Sort parameters
$Request = getReqParams();
// Notification password
$NOTIFY_PWD = "***";
// Get sign
$reqres = checkSign($NOTIFY_PWD, $Request);

// Get sign from the request
$SIGN_REQ = getSign();

if ($reqres == $SIGN_REQ) {
    $error = 0;
}
else $error = 151;

//Response
header('Content-Type: text/xml');
$xmlres = <<<XML
<?xml version="1.0"?>
<result>
<result_code>$error</result_code>
</result>
XML;
echo $xmlres;
?>
~~~

# Responses

Response may be in XML or JSON format, depending on "Accept" header in request.

## Successful Invoice Operations {#response_bill}

Object with operation's result.

<aside class="notice">
When you send two consecutive requests with the same <i>{prv_id}</i>, <i>{bill_id}</i> and <i>amount</i> parameters, you will get the same result code.
</aside>
	 
~~~xml
<response>
   <result_code>0</result_code>
   <bill>
    <bill_id>bill1234</bill_id>
    <amount>99.95</amount>
    <originAmount>99.95</originAmount>
    <ccy>RUB</ccy>
    <originCcy>RUB</originCcy>
    <ccy>USD</ccy>
    <status>paid<status>
    <error>0</error>
    <user>tel:+79161231212</user>
    <comment>Invoice from ShopName</comment>
   </bill>
</response>
~~~

~~~json
{
 "response": {
  "result_code": 0,
  "bill": {
    "bill_id": "BILL-1",
    "amount": "10.00",
    "originAmount": "10.00", 
    "ccy": "RUB",
    "originCcy": "RUB",
    "status": "waiting",
    "error": 0,
    "user": "tel:+79031234567",
    "comment": "test"
  }
}}
~~~

Parameter|Type|Description
--------|---|--------
result_code|Integer|Only "0", means successful operation
bill_id|String|Unique invoice identifier generated by the merchant
amount|String|The invoice amount. The rounding up method depends on the invoice currency.
originAmount|String|The invoice amount in the original invoice's currency (see `originCcy` parameter). The rounding up method depends on the invoice currency.
ccy	|String|Currency identifier (Alpha-3 ISO 4217 code)
originCcy|String|Currency identifier of the invoice (Alpha-3 ISO 4217 code)
status	|String|Current [invoice status](#status)
error	|Integer|[Error code](#errors)
user|String|The Visa QIWI Wallet user’s ID, to whom the invoice is issued. It is the user’s phone number with "tel:" prefix.
comment|String|Comment to the invoice

## Successful Refund Operations {#response_refund}

Object with operation's result.
	 
~~~xml
<response>
<result_code>0</result_code>
<refund>
<refund_id>122swbill</refund_id>
<amount>10.0</amount>
<status>processing<status>
<error>0</error>
<user>tel:+79161231212</user>
</refund>
</response>
~~~

~~~json
{
 "response": {
  "result_code": 0,
  "refund": {
    "refund_id": 122swbill,
    "amount": "10.0",
    "status": "processing",
    "error": 0,
    "user": "tel:+79161231212"
  }
}}
~~~

Parameter|Type|Description
--------|---|--------
result_code|Integer|Only "0", means successful operation
refund_id|String|The refund identifier, unique number in a series of refunds processed for a particular invoice
amount|String|The actual amount of the refund. The rounding up method depends on the original invoice currency.
status	|String|Current [refund status](#status_refund)
error	|Integer|[Error code](#errors). **Important!** When the amount of refund exceeds the initial invoice amount or the amount left after the previous refunds, error code 242 is returned.
user|String|The Visa QIWI Wallet user’s ID, to whom the original invoice is issued. It is the user’s phone number with "tel:" prefix.

## Response in Case of Operation's Error {#response_error}

There will be no invoice/refund data in the response, when corresponding operation is unsuccessful.

~~~xml
<response>
<result_code>150</result_code>
<description>Authorization failed</description>
</response>
~~~

~~~json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

Parameter|Type|Description
--------|---|--------
result_code|Integer|[Result code of the operation](#errors)
description|String|Error description

# Operation Statuses

## Invoice Status {#status}

Status|Description|Final
------|--------|---------
waiting | Invoice issued, pending payment| N
paid|Invoice has been paid|Y
rejected|Invoice has been rejected|Y
unpaid|Payment processing error. Invoice has not been paid|Y
expired	|Invoice expired. Invoice has not been paid|Y

## Refund Status {#status_refund}

Status|Description|Final
------|--------|---------
processing | Payment refund is pending| N
success|Payment refund is successful|Y
fail|Payment refund is unsuccessful|Y

# Error Codes {#errors}

<aside class="notice"><b>Fatal</b> means the result will not change with the second and subsequent requests (error is not temporary)</aside> 

Code| Description |Fatal
---|----------|------
0|Success	| Unrelated
5|Incorrect data in the request parameters|Y
13|Server is busy, try again later|N
78|Operation is forbidden|Y
150|Authorization error (e.g. invalid login/password)|Y
152|Protocol is not enabled or protocol is disabled|N
155|This merchant’s identifier ([API ID](#auth)) is blocked|Y
210|Invoice not found|Y
215|Invoice with this `bill_id` already exists|Y
241|Invoice amount is less than allowed|Y
242|Invoice amount is greater than allowed. Also returns to refund request when the amount of refund exceeds the initial invoice amount or the amount left after the previous refunds | Y
298|User not registered|Y
300|Technical error|N
303|Wrong phone number|Y
316|Authorization from the blocked merchant|N
319|No rights for the operation|N
339|IP-addresses blocked|Y
341|Required parameter is incorrectly specified or absent in the request|Y
700|Monthly limit on operations is exceeded|Y
774|Visa QIWI Wallet user account temporarily blocked|Y
1001|Currency is not allowed for the merchant|Y
1003|No convert rate for these currencies|N
1019|Unable to determine wireless operator for MNO balance payment|Y
1419|Bill was already payed|Y

