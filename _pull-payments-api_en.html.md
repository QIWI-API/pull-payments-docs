# Pull REST API {#pull_rest_api}

###### Last update: 2022-05-03 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_pull-payments-api_en.html.md)

## Invoicing Operation Flow

![Pull API Invoicing](/images/pullrest_1_en.png)

* User submits an order on the merchant’s website.

* Merchant service [issues invoice](#invoice).

* To increase successful payments conversion, merchant is recommended to redirect the user to [QIWI Checkout](#checkout) page on QIWI Wallet site, to pay for the invoice. Otherwise, invoice can be paid in any QIWI Wallet interfaces, such as web (qiwi.com), mobile applications and self-service terminals.

* If the merchant enables [notifications](#notification), then its service receives a notification from QIWI Wallet system once invoice is paid or cancelled by the user. Authorization on the merchant's side is required for notifications.

* In any case, merchant can [request current status](#invoice-status) of the invoice, or [cancel invoice](#cancel) (provided that the user has not initiated payment yet).

* When the invoice payment is confirmed, merchant delivers ordered services/goods.

<aside class="notice">
Invoicing operation on Pull REST API is idempotent, so on two consecutive requests with the same <i>prv_id</i>, <i>bill_id</i> and <i>amount</i> parameters the same response code is returned.
</aside>

## Developer Tools {#node_sdk}

* [NODE JS SDK](https://github.com/QIWI-API/pull-payments-node-js-sdk) - Node JS package of ready-to-use solutions for server2server integration development to begin accepting payments on your site.

## Authorization {#auth_param}

Pull REST API requests are authorized through HTTP Basic-authorization with API ID and API password. Header is `Authorization` string and its value is `Basic Base64(API_ID:API_PASSWORD)`.

~~~shell
user@server:~$ curl "server_URL"
  --header "Authorization: Basic MjMyNDQxMjM6NDUzRmRnZDQ0Mw=="
~~~

<ul class="nestedList params">
    <li><h3>Authorization and form data</h3><span>Data can be obtained on <a href="https://kassa.qiwi.com">kassa.qiwi.com</a></span>
    </li>
</ul>

Parameter|Description|Type|Required
 ---------|--------|---|------
 API_ID | Provider API identifier for authorization | Integer| +
 API_PASSWORD | API password for authorization| String | +
 Shop ID | Numeric identifier of the provider's service | Integer | +

<aside class="notice">
To obtain service data, open "Protocols - REST - Authorization" section on <a href='http://kassa.qiwi.com'>kassa.qiwi.com</a>.

<ul class="nestedList notice_image">
    <li><h3>Details</h3>
        <ul>
             <li><img src="/images/pull_rest_auth_kassa.png" /></li>
        </ul>
    </li>
</ul>

</aside>

## Issuing Invoice {#invoice}

Creates new invoice to the specified phone number (wallet ID in QIWI Wallet).

<h3 class="request method">Request → PUT</h3>

~~~php
<?php
//Example
//Shop identifier from Merchant details page
$SHOP_ID = "21379721";
//API ID from Merchant details page
$REST_ID = "62573819";
//API password from Merchant details page
$PWD = "**********";
//Invoice ID
$BILL_ID = "99111-ABCD-1-2-1";
$PHONE = "79191234567";

$data = array(
    "user" => "tel:+" . $PHONE,
    "amount" => "1000.00",
    "ccy" => "RUB",
    "comment" => "Good purchase",
    "lifetime" => "2015-01-30T15:35:00",
    "pay_source" => "qw",
    "prv_name" => "Special packages"
);

$ch = curl_init('https://api.qiwi.com/api/v2/prv/'.$SHOP_ID.'/bills/'.$BILL_ID);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'PUT');
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($data));
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
curl_setopt($ch, CURLOPT_HTTPAUTH, CURLAUTH_BASIC);
curl_setopt($ch, CURLOPT_USERPWD, $REST_ID.":".$PWD);
curl_setopt($ch, CURLOPT_HTTPHEADER,array (
    "Accept: application/json"
));
$results = curl_exec ($ch) or die(curl_error($ch));
echo $results;
echo curl_error($ch);
curl_close ($ch);
//Optional user redirect
$url = 'https://oplata.qiwi.com/order/external/main.action?shop='.$SHOP_ID.'&
transaction='.$BILL_ID.'&successUrl=http%3A%2F%2Fexample.com%2Findex.php%3Froute%3D
payment%2Fqiwi%2Fsuccess&failUrl=http%3A%2F%2Fexample.com%2Findex.php%3Froute%3D
payment%2Fqiwi%2Ffail&pay_source=card';
echo '<br><br><b><a href="'.$url.'">Redirect link to pay for invoice</a></b>';
?>
~~~

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/BILL-1"
  -X PUT  --header "Accept: text/json" --header "Authorization: Basic ***"
  -d "user=tel%3A%2B79161234567&amount=10.00&ccy=RUB&comment=test&lifetime=2016-09-25T15:00:00"
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Parameters are in the PUT-request URL pathname:</strong>
             <li><strong>prv_id</strong> - merchant’s Shop ID (numeric value, as displayed in "Shop ID" parameter of "Protocols Settings" section of kassa.qiwi.com web site)</li>
             <li><strong>bill_id</strong> - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters). Uniqueness means that the identifier must not coincide with any of previously issued invoices of the merchant.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - JSON response</li>
             <li>Accept: text/xml or Accept: application/xml - XML response</li>
             <li>Content-type: application/x-www-form-urlencoded; charset=utf-8</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Parameters are sent in the request body as <i>formdata</i>.</span>
    </li>
</ul>

Parameter|Description|Type|Required
---------|--------|---|------
user | The QIWI Wallet user’s ID, to whom the invoice is issued. It is the user’s phone number with `tel:` prefix | String(20)|Y
amount | The invoice amount. The number is rounded down with two decimal places | Number(6.2)|Y
ccy | Invoice currency identifier (Alpha-3 ISO 4217 code). Depends on currencies allowed for the merchant. The following values are supported: RUB, EUR, USD, KZT | String(3)|Y
comment | Comment to the invoice | String(255)|Y
lifetime | Date and time up to which the invoice is available for payment, URL-encoded ISO 8601 (`YYYY-MM-DDThh:mm:ss`), Moscow timezone. If the invoice is not paid by this date it will become void and will be assigned a final status.<br> **Important! Invoice will be automatically expired when 45 days is passed after the invoicing date**|dateTime|Y
pay_source |If the value is `mobile` the user’s MNO balance will be used as a funding source. If the value is `qw`, any other funding source is used available in QIWI Wallet system for the user. If parameter isn’t present, value `qw` is assumed |String |N
prv_name|Merchant’s name| String(100)|N

The given PHP example implements creation of the invoice and redirection to the QIWI Checkout web page for payment. This example demonstrates using merchant's [authorization parameters](#auth_param), i.e. shop ID, API ID and password for the API ID. Open the _PHP_ tab on the right.

<h3 class="request">Response ←</h3>

~~~http

HTTP/1.1 200 OK
Content-Type: text/json
~~~

~~~json
{
  "response": {
     "result_code": 0,
     "bill": {
        "bill_id": "BILL-1",
        "amount": "10.00",
        "ccy": "RUB",
        "status": "waiting",
        "error": 0,
        "user": "tel:+79031234567",
        "comment": "My comment"
     }
  }
}
~~~

~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/json
~~~

~~~json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

~~~http

HTTP/1.1 200 OK
Content-Type: text/xml
~~~

~~~xml
<response>
   <result_code>0</result_code>
   <bill>
    <bill_id>BILL-1</bill_id>
    <amount>10.00</amount>
    <originAmount>10.00</originAmount>
    <ccy>RUB</ccy>
    <originCcy>RUB</originCcy>
    <status>rejected<status>
    <error>0</error>
    <user>tel:+79161234567</user>
    <comment>test</comment>
   </bill>
</response>
~~~

~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/xml
~~~

~~~xml
<response>
   <result_code>341</result_code>
   <description>Authorization is failed</description>
</response>
~~~

Response format depends on `Accept` header in the request:

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - response in JSON</li>
             <li>Accept: text/xml or Accept: application/xml - response in XML</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Parameter|Type|Description
--------|---|--------
result_code|Integer|[Error code](#errors)
description|String|Error description. Returned when `result_code` is non-zero.
bill|Object|Bill data. Returned when `result_code` is zero (successful operation). Parameters:
bill.bill_id|String|Unique invoice identifier generated by the merchant
bill.amount|String|The invoice amount. The number is rounded down with two decimal places.
bill.ccy|String|Currency identifier of the invoice (Alpha-3 ISO 4217 code)
bill.status|String|Current [invoice status](#status)
bill.error|Integer|Always `0`, means successful operation
bill.user|String|The QIWI Wallet user’s ID, to whom the invoice is issued. It is the user’s phone number with `tel:` prefix.
bill.comment|String|Comment to the invoice

## Checking Invoice Status {#invoice-status}

Gets payment status of the invoice.

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/sdf23452435"
  --header "Authorization: Basic ***" --header "Accept: text/json"
~~~

<h3 class="request method">Request → GET</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Parameters are in the GET-request URL pathname:</strong>
             <li><strong>prv_id</strong> - merchant’s Shop ID (numeric value, as displayed in "Shop ID" parameter of "Protocols Settings" section of kassa.qiwi.com web site)</li>
             <li><strong>bill_id</strong> - invoice identifier</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - JSON response</li>
             <li>Accept: text/xml or Accept: application/xml - XML response</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http

HTTP/1.1 200 OK
Content-Type: text/json
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
        "comment": "My comment"
     }
  }
}
~~~

~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/json
~~~

~~~json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

~~~http

HTTP/1.1 200 OK
Content-Type: text/xml
~~~

~~~xml
<response>
   <result_code>0</result_code>
   <bill>
    <bill_id>BILL-1</bill_id>
    <amount>10.00</amount>
    <originAmount>10.00</originAmount>
    <ccy>RUB</ccy>
    <originCcy>RUB</originCcy>
    <status>rejected<status>
    <error>0</error>
    <user>tel:+79161234567</user>
    <comment>test</comment>
   </bill>
</response>
~~~

~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/xml
~~~

~~~xml
<response>
   <result_code>341</result_code>
   <description>Authorization is failed</description>
</response>
~~~

Response format depends on `Accept` header in the request:

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - response in JSON</li>
             <li>Accept: text/xml or Accept: application/xml - response in XML</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Parameter|Type|Description
--------|---|--------
result_code|Integer|[Error code](#errors)
description|String|Error description. Returned when `result_code` is non-zero.
bill|Object|Bill data. Returned when `result_code` is zero (successful operation). Parameters:
bill.bill_id|String|Unique invoice identifier generated by the merchant
bill.amount|String|The invoice amount. The number is rounded down with two decimal places.
bill.originAmount|String|The amount taken from the balance when the invoice get paid (see `originCcy` parameter). The number is rounded down with two decimal places. **Returns for invoices when the user initiates payment**.
bill.ccy|String|Currency identifier of the invoice (Alpha-3 ISO 4217 code)
bill.originCcy|String|Currency identifier of the balance from which the invoice is paid (Alpha-3 ISO 4217 code). **Returns for invoices when the user initiates payment**.
bill.status|String|Current [invoice status](#status)
bill.error|Integer|Always `0`, means successful operation
bill.user|String|The QIWI Wallet user’s ID, to whom the invoice is issued. It is the user’s phone number with `tel:` prefix.
bill.comment|String|Comment to the invoice

## Cancelling Unpaid Invoice {#cancel}

Cancels unpaid invoice provided that its lifetime has not expired yet.

~~~shell
user@server:~$ curl -X PATCH
  --header "Authorization: Basic ***"  --header "Accept: text/json"
  "https://api.qiwi.com/api/v2/prv/373712/bills/BILL-1"
  -d "status=rejected"
~~~

<h3 class="request method">Request → PATCH</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Parameters are in the PATCH-request URL pathname:</strong>
             <li><strong>prv_id</strong> - merchant’s Shop ID (numeric value, as displayed in "Shop ID" parameter of "Protocols Settings" section of kassa.qiwi.com web site)</li>
             <li><strong>bill_id</strong> - invoice identifier</li>
        </ul>
         <ul>
         <strong>Parameter is in the request body.</strong>
             <li><strong>status</strong> - <i>rejected</i> (cancelling status).</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - JSON response</li>
             <li>Accept: text/xml or Accept: application/xml - XML response</li>
             <li>Content-type: application/x-www-form-urlencoded; charset=utf-8</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http

HTTP/1.1 200 OK
Content-Type: text/json
~~~

~~~json
{
  "response": {
     "result_code": 0,
     "bill": {
        "bill_id": "BILL-1",
        "amount": "10.00",
        "ccy": "RUB",
        "status": "rejected",
        "error": 0,
        "user": "tel:+79031234567",
        "comment": "My comment"
     }
  }
}
~~~

~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/json
~~~

~~~json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

~~~http

HTTP/1.1 200 OK
Content-Type: text/xml
~~~

~~~xml
<response>
   <result_code>0</result_code>
   <bill>
    <bill_id>BILL-1</bill_id>
    <amount>10.00</amount>
    <ccy>RUB</ccy>
    <status>rejected<status>
    <error>0</error>
    <user>tel:+79161234567</user>
    <comment>test</comment>
   </bill>
</response>
~~~

~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/xml
~~~

~~~xml
<response>
   <result_code>341</result_code>
   <description>Authorization is failed</description>
</response>
~~~

Response format depends on `Accept` header in the request:

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - response in JSON</li>
             <li>Accept: text/xml or Accept: application/xml - response in XML</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Parameter|Type|Description
--------|---|--------
result_code|Integer|[Error code](#errors)
description|String|Error description. Returned when `result_code` is non-zero.
bill|Object|Bill data. Returned when `result_code` is zero (successful operation). Parameters:
bill.bill_id|String|Unique invoice identifier generated by the merchant
bill.amount|String|The invoice amount. The number is rounded down with two decimal places.
bill.ccy|String|Currency identifier of the invoice (Alpha-3 ISO 4217 code)
bill.status|String|Rejected [invoice status](#status)
bill.error|Integer|Always `0`, means successful operation
bill.user|String|The QIWI Wallet user’s ID, to whom the invoice is issued. It is the user’s phone number with `tel:` prefix.
bill.comment|String|Comment to the invoice

## Refunds {#refund}

Method processes a full or partial refund to user's QIWI Wallet account, so a reversed transaction with the same currency is created for the initial one.

Merchant can create several refund operations for the same initial invoice provided that:

* Amount of all refund operations does not exceed initial invoice amount.
* Different refund IDs used for different refund operations of the same invoice (see below).

<aside class="warning">
When the transmitted amount exceeds the initial invoice amount or the amount left after the previous refunds, server returns error code 242.
</aside>

### Refund Operation Flow

![Refund Invoice REST API](/images/pullrest_2_en.png)

* Merchant sends a request for refund to QIWI Wallet server.
* To make sure that the payment refund has been successfully processed, merchant can periodically request the invoice [refund status](#refund_status) until the final status is received.
* This scenario can be repeated multiple times until the invoice is completely refunded (whole invoice amount has been returned to the user).

<h3 class="request method">Request → PUT</h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/BILL-1/refund/REF1"
  -v -w "%{http_code}"
  -X PUT  --header "Accept: text/json"
  --header "Authorization: Basic ***"
  --header "Content-type: application/x-www-form-urlencoded; charset=utf-8"
  -d "amount=5.0"
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a>/refund/<a>refund_id</a></span></h3>
        <ul>
        <strong>Parameters are in the URL pathname:</strong>
             <li><strong>prv_id</strong> - merchant’s Shop ID (numeric value, as displayed in "Shop ID" parameter of "Protocols Settings" section of kassa.qiwi.com web site)</li>
             <li><strong>bill_id</strong> - identifier of the invoice to be refunded</li>
             <li><strong>refund_id</strong> - the refund identifier, a number specific to a series of refunds for the invoice <i>{bill_id}</i> (string of 1 to 9 symbols – any 0-9 digits and upper/lower Latin letters)</li>
        </ul>
         <ul>
         <strong>Parameter is in the request body:</strong>
             <li><strong>amount</strong> - the refund amount. It should be less or equal to the amount of the original invoice specified in <b>bill_id</b>. The positive number that is rounded down with two decimal places.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - JSON response</li>
             <li>Accept: text/xml or Accept: application/xml - XML response</li>
             <li>Content-type: application/x-www-form-urlencoded; charset=utf-8</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http

HTTP/1.1 200 OK
Content-Type: text/json
~~~

~~~json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": "REF1",
         "amount": "5.00",
         "status": "success",
         "error": 0
      }
   }
}
~~~

~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/json
~~~

~~~json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

~~~http

HTTP/1.1 200 OK
Content-Type: text/xml
~~~

~~~xml
<response>
  <result_code>0</result_code>
  <refund>
   <refund_id>REF1</refund_id>
   <amount>5.0</amount>
   <status>success<status>
   <error>0</error>
  </refund>
</response>
~~~

~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/xml
~~~

~~~xml
<response>
   <result_code>341</result_code>
   <description>Authorization is failed</description>
</response>
~~~

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - response in JSON</li>
             <li>Accept: text/xml or Accept: application/xml - response in XML</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Parameter|Type|Description
--------|---|--------
result_code|Integer|[Error code](#errors)
description|String|Error description. Returned when `result_code` is non-zero.
refund|Object|Refund data. Returned when `result_code` is zero (successful operation). Parameters:
refund.refund_id|String|The refund identifier, unique number in a series of refunds processed for a particular invoice
refund.amount|String|The actual amount of the refund. The positive number that is rounded down with two decimal places.
refund.status|String|Current [refund status](#status_refund)
refund.error|Integer|[Error code](#errors).<br>**Important!** When the amount of refund exceeds the initial invoice amount or the amount left after the previous refunds, error code `242` is returned.

## Check Refund Status

Method returns current status of the refund.

<h3 class="request method">Request → GET</h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/BILL-1/refund/REF1"
  -v -w "%{http_code}"
  --header "Accept: text/json" --header "Authorization: Basic ***"
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a>/refund/<a>refund_id</a></span></h3>
        <ul>
        <strong>Parameters are in the URL pathname:</strong>
             <li><strong>prv_id</strong> - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of kassa.qiwi.com web site)</li>
             <li><strong>bill_id</strong> - invoice identifier</li>
             <li><strong>refund_id</strong> - refund identifier, a number specific to a series of refunds for the invoice <i>{bill_id}</i> (string of 1 to 9 symbols – any 0-9 digits and upper/lower Latin letters)</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - JSON response</li>
             <li>Accept: text/xml or Accept: application/xml - XML response</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http

HTTP/1.1 200 OK
Content-Type: text/json
~~~

~~~json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": "REF1",
         "amount": "5.00",
         "status": "success",
         "error": 0
      }
   }
}
~~~

~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/json
~~~

~~~json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

~~~http

HTTP/1.1 200 OK
Content-Type: text/xml
~~~

~~~xml
<response>
  <result_code>0</result_code>
  <refund>
   <refund_id>REF1</refund_id>
   <amount>5.0</amount>
   <status>success<status>
   <error>0</error>
  </refund>
</response>
~~~

~~~http

HTTP/1.1 401 Unauthorized
Content-Type: text/xml
~~~

~~~xml
<response>
   <result_code>341</result_code>
   <description>Authorization is failed</description>
</response>
~~~

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - response in JSON</li>
             <li>Accept: text/xml or Accept: application/xml - response in XML</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Parameter|Type|Description
--------|---|--------
result_code|Integer|[Error code](#errors)
description|String|Error description. Returned when `result_code` is non-zero.
refund|Object|Refund data. Returned when `result_code` is zero (successful operation). Parameters:
refund.refund_id|String|The refund identifier, unique number in a series of refunds processed for a particular invoice
refund.amount|String|The actual amount of the refund. The positive number that is rounded down with two decimal places.
refund.status|String|Current [refund status](#status_refund)
refund.error|Integer|[Error code](#errors). **Important!** When the amount of refund exceeds the initial invoice amount or the amount left after the previous refunds, error code `242` is returned.

## Operation Statuses {#status}

### Invoice Status {#status_bills}

Status|Description|Final
------|--------|---------
waiting | Invoice issued, pending payment| N
paid|Invoice has been paid|Y
rejected|Invoice has been rejected|Y
unpaid|Payment processing error. Invoice has not been paid|Y
expired |Invoice expired. Invoice has not been paid|Y

### Refund Status {#status_refund}

Status|Description|Final
------|--------|---------
processing | Payment refund is pending| N
success|Payment refund is successful|Y
fail|Payment refund is unsuccessful|Y

## Error Codes {#errors}

<aside class="notice"><b>Fatal</b> means the result will not change with the second and subsequent requests (error is not temporary)</aside>

Code| Description |Fatal
---|----------|------
0|Success   | Unrelated
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
774|QIWI Wallet user account temporarily blocked|Y
934|Region is not supported
1001|Currency is not allowed for the merchant|Y
1003|No convert rate for these currencies|N
1018|Country is not supported
1019|Unable to determine wireless operator for MNO balance payment|Y
1419|Bill was already payed|Y
