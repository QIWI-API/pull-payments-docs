# Online Invoicing Web Form {#webform_ru}

###### Last update: 2017-07-11 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_webform_en.html.md)


When calling the web form, server displays Visa QIWI Wallet invoice web page (or page with error result). With appropriate parameters set, client can input a phone number and an invoice amount for invocing on the web form directly.

Invoice number is automatically assigned in Visa QIWI Wallet system. Client can pay for invoice on the same page immediately as automatic redirection to the [QIWI Checkout](#checkout_en) page occurs.

* Web form can be called in two ways:
    * [WF](#wf) - without merchant's authorization.
    * [WF/auth](#wfauth) - with merchant's authorization (by its identifier and the requests' digital signature). It is currently in its beta version. To use the protocol, please address to QIWI Technical Support (bss@qiwi.ru).

## Operations flow

1. User submits an order on the merchant’s website. 
2. Merchant calls invoicing web form. A call may use authorization parameters.
3. When invoicing was successfulll, web form redirect the user to [QIWI Checkout](#checkout_en) page.
4. If merchant enables [notifications](#notification_en), then Visa QIWI Wallet sends to the merchant's server a notification on the invoice status once invoice is paid or cancelled by the user. Authorization on the merchant's side is required for notifications.
6. Merchant delivers ordered services/goods when the invoice gets paid.

<h3 class="request method">Request → REDIRECT</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://bill.qiwi.com/order/external/create.action</span></h3></li>
</ul>

## Web form without authorization  {#wf}

~~~text
  Without authorization
~~~

~~~http
GET /order/external/create.action?comm=test&txn_id=0000&from=000000&summ=1.11&successUrl=http%3A%2F%2Ftest.ru%3Fcurrency=643&to=%2B71234567890 HTTP/1.1
Host: bill.qiwi.com
~~~

<ul class="nestedList example">
    <li><h3>No authorization from merchant</h3><span><a>https://bill.qiwi.com/order/external/create.action?comm=test&txn_id=0000&from=000000&summ=1.11&successUrl=http%3A%2F%2Ftest.ru%3Fcurrency=643&to=%2B71234567890</a></span>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Invoice parameters are specified in the web form URL.</span></li>
</ul>

Parameter|Description|Type|Required
---------|--------|---|---
from | Merchant identifier ([Shop ID](#auth_param)). Identifier is specified in "HTTP-protocol" part of Settings section of merchant's account on ishop.qiwi.com.|Integer|+
currency | Invoice currency identifier (in Alpha-3 ISO 4217 code). Any currency may be used if specified in agreement with Visa QIWI Wallet. | String(3)|+
to | Visa QIWI Wallet client phone number to make the invoice. If not specified, client should enter the phone on the web invoice form.| String(20)|-
summ | Amount of the invoice. Positive number rounded to 2 or 3 fractional digits. Point as a separator. If not specified, client should enter the amount on the web invoice form.| Number(6.3)|-
txn_id|Unique invoice number in online merchant store. It is used to identify specific invoice of the merchant.|String(30), Latin letters and digits (without space)|-
comm | Merchant commentary to the invoice. If not specified and `to` parameter is absent, client may enter the comment with phone number at once on the web form. | String(255)|-
lifetime | Invoice lifetime (in minutes), counting from invoice creation time. When time is expired, invoice cannot be paid and it would be cancelled. **Important! Invoice will be automatically expired when 45 days is passed after the invoicing date.**|-
iframe| This parameter (if `true`) means that invoice page would be opened in "iframe". Page view is tight and easily framed into merchant's site.|Boolean, `true`/`false`<br>`false` by default|-
successUrl|The URL to which the user will be redirected in case of successful creation of Visa QIWI Wallet transaction. May be parameter or anchor. URL must be within merchant's site. Redirection is performed when user pays by Visa QIWI Wallet balance from Visa QIWI Wallet site only.|URL-encoded string|-
failUrl|The URL to which the user will be redirected in case of successful creation of Visa QIWI Wallet transaction Client is redirected to the specified URL when Visa QIWI Wallet transaction creation is unsuccessful. May be parameter or anchor. URL must be within merchant's site. Redirection is performed when user pays by Visa QIWI Wallet balance from Visa QIWI Wallet site only. |URL-encoded string|-
target|This parameter means that hyperlink specified in `successUrl` / `failUrl` parameter opens in "iframe". Void if absent|String (`iframe` only)|-
pay_source |Default payment method to show first for the client on QIWI Checkout. Possible values:<br>`qw` – Visa QIWI Wallet account;<br> `mobile` – client’s cell phone account;<br> `card` – a credit/debit card;<br> `wm` – WebMoney wallet if linked to Visa QIWI Wallet account; <br> `ssk` – payment by cash in a QIWI Terminal.<br>ЕWhen specified method is inaccessible, the page contains notice about it and the client can choose another method. |String (predefined)|-

## Web form with authorization {#wfauth}

~~~text
  With authorization
~~~

~~~http
GET /order/external/create.action?from=260831&txn_id=q115928&summ=1.12&api_id=46835183&currency=RUB&sign=7d3e8b8df7c8ed70b1089c6a5ae86eddd1ee HTTP/1.1
Host: bill.qiwi.com
~~~

<ul class="nestedList example">
    <li><h3>Merchant's authorization</h3><span><a>https://bill.qiwi.com/order/external/create.action?from=260831&txn_id=q115928&summ=1.12&api_id=46835183&currency=RUB&sign=7d3e8b8df7c8ed70b1089c6a5ae86eddd1ee</a></span>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Invoice parameters are specified in the web form URL.</span></li>
</ul>

Parameter|Description|Type|Required
---------|--------|---|------
from | Merchant identifier ([Shop ID](#auth_param)). Identifier is specified in "HTTP-protocol" part of Settings section of merchant's account on ishop.qiwi.com.|Integer|+
currency | Invoice currency identifier (in Alpha-3 ISO 4217 code). Any currency may be used if specified in agreement with Visa QIWI Wallet. | String(3)|+
to | Visa QIWI Wallet client phone number to make the invoice. If not specified, client should enter the phone on the web invoice form. **Important! Parameter is not included into the request signature** | String(20)|-
summ | Amount of the invoice. Positive number rounded to 2 or 3 fractional digits. Point as a separator. | Number(6.3)|+
txn_id|Unique invoice number in online merchant store. It is used to identify specific invoice of the merchant.|String(30)|+
api_id|[API_ID](#auth_param)|Integer|+
sign|[Request signature](#http_sign)|String|+
comm | Merchant commentary to the invoice. If not specified and `to` parameter is also void, client may enter the comment with phone number at once on the web form. | String(255)|-
lifetime | Date/time when invoice will be expired. After that date, invoice cannot be paid and it would be expired. Maximum period before invoice expiration is 45 days. When parameter is greater, then expiration date would set to 45 days. Parameter is included into the [request signature](#http_sign). Important! If `lifetime` parameter is in wrong format or missing, invoice will be automatically expired when 28 days are passed from the invoice creation.|YYYY-MM-DDTHHMM|-
iframe| This parameter (if true) means that invoice page would be opened in "iframe". Page view is tight and easily framed into merchant's site.|Boolean, `true`/`false`<br>`false` by default|-
successUrl|The URL to which the user will be redirected in case of successful creation of Visa QIWI Wallet transaction. May be parameter or anchor. URL must be within merchant's site. Redirection is performed when user pays by Visa QIWI Wallet balance from Visa QIWI Wallet site only.|URL-encoded string|-
failUrl|The URL to which the user will be redirected in case of successful creation of Visa QIWI Wallet transaction Client is redirected to the specified URL when Visa QIWI Wallet transaction creation is unsuccessful. May be parameter or anchor. URL must be within merchant's site. Redirection is performed when user pays by Visa QIWI Wallet balance from Visa QIWI Wallet site only. |URL-encoded string|-
target|This parameter means that hyperlink specified in `successUrl` / `failUrl` parameter opens in "iframe". Void if absent|String (`iframe` only)|-
pay_source |Default payment method to show first for the client on QIWI Checkout. Possible values:<br>`qw` – Visa QIWI Wallet account;<br> `mobile` – client’s cell phone account;<br> `card` – a credit/debit card;<br> `wm` – WebMoney wallet if linked to Visa QIWI Wallet account; <br> `ssk` – payment by cash in a QIWI Terminal.<br>ЕWhen specified method is inaccessible, the page contains notice about it and the client can choose another method. |String|-

### Request signature {#http_sign}

Online web form invoicing with authorization requires a signature of the request (`sign` parameter).

<aside class="notice">
To get signature, use <a href='#auth_param'>API_PASSWORD</a> – a password generated with corresponding API ID (<i>api_id</i> parameter of the request). 
</aside>

Signature algorithm follows:

1. Get a string with values of all required parameters of the GET-request and `lifetime` parameter (if exists) in alphabetical order separated by `|` symbol:

    `Invoice_parameters = "{api_id}|{currency}|{from}|{lifetime}|{summ}|{txn_id}"`
    Where `{parameter}` is a value of the corresponding parameter.

2. Calculate HMAC-hash with SHA256-encryption on the obtained string using API password (password to the given API ID) taken as a key:
    `sign = HMAС(SHA256, API_password, Invoice_parameters)`

