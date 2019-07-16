# Online Invoicing Web Form {#webform_en}

###### Last update: 2017-11-15 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_webform_en.html.md)

Client receives a web form with selection of appropriate payment methods for the invoice.

Web form calling is performed without merchant's authorization. Client can input a phone number and an amount for invoicing on the web form directly.

## Operations flow {#webform_flow}

* User submits an order on the merchant’s website.

* Merchant calls invoicing web form. When calling is without a phone number, a user enters the number on the web form directly.

* When invoicing was successful, the user is immediately redirected to [QIWI Checkout](#checkout_en) page.

* If merchant enables [notifications](#notification_en), then QIWI Wallet sends to the merchant's server a notification on the invoice status once invoice is paid or cancelled by the user. Authorization on the merchant's side is required for notifications.

* Merchant delivers ordered services/goods when the invoice payment is confirmed.

<h3 class="request method">Request → REDIRECT</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://bill.qiwi.com/order/external/create.action</span></h3></li>
</ul>

~~~http
GET /order/external/create.action?txn_id=10000&from=11223&summ=1.11&currency=643 HTTP/1.1
Host: bill.qiwi.com
~~~

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Invoice parameters are specified in the web form URL.</span></li>
</ul>

Parameter|Description|Type|Required
---------|--------|---|---
from | Merchant identifier ([Shop ID](#auth_param)). Identifier is specified in "HTTP-protocol" part of "Settings" section of merchant's account on kassa.qiwi.com.|Integer|+
currency | Invoice currency identifier (in Alpha-3 ISO 4217 code). Any currency may be used if specified in agreement with QIWI Wallet. | String(3)|+
to | QIWI Wallet client phone number to make the invoice. If not specified, client should enter the phone on the web invoice form.| String(20)|-
summ | Amount of the invoice. Positive number rounded to two fractional digits. Point as a separator. If not specified, client should enter the amount on the web invoice form.| Number(6.2)|-
txn_id|Unique invoice number in online merchant store. It is used to identify specific invoice of the merchant.|String(30), Latin letters and digits (without space)|-
comm | Merchant commentary to the invoice. If not specified and `to` parameter is absent, client may enter the comment with phone number at once on the web form. | String(255)|-
lifetime | Invoice expiry date (`YYYY-MM-DDTHHMM` format). When time is expired, invoice cannot be paid and it would be cancelled. **Important! Invoice will be automatically expired when 28 days is passed after the invoicing date.**|-
successUrl|The URL to which the user [will be redirected](#back_url) in case of successful creation of QIWI Wallet transaction. May be parameter or anchor. URL must be within merchant's site. **Redirection is performed when user pays by QIWI Wallet account's balance only.**|URL-encoded string|-
failUrl|The user [is redirected](#back_url) to the specified URL when QIWI Wallet transaction creation is unsuccessful. May be parameter or anchor. URL must be within merchant's site. **Redirection is performed when user pays by QIWI Wallet account's balance only.** |URL-encoded string|-
pay_source |Default payment method to show first for the client on QIWI Checkout. Possible values:<br>`qw` – QIWI Wallet account;<br> `mobile` – client’s cell phone account;<br> `card` – a credit/debit card;<br> `wm` – WebMoney wallet if linked to QIWI Wallet account; <br> `ssk` – payment by cash in a QIWI Terminal.<br>When specified method is inaccessible, the page contains notice about it and the client can choose another method. |String (predefined)|-
