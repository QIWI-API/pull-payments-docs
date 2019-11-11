# Notifications {#notification_en}

###### Last update: 2017-11-14 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_notification_en.html.md)

Notification is a POST-request (callback). The request's body contains all relevant data of the invoice serialized as HTTP-request parameters and encoded by UTF-8 plus parameter `command=bill`.

<h3 class="request method">Request → POST</h3>

> Example

~~~shell
user@server:~$ curl "https://service.ru/qiwi-notify.php"
  -v -w "%{http_code}"
  -X POST --header "Accept: text/xml"
  --header "Content-Type: application/x-www-form-urlencoded; charset=utf-8"
  --Authorization: "Basic MjA0Mjp0ZXN0Cg=="
  -d "bill_id=BILL-1%26status=paid%26amount=1.00%26user=tel%3A%2B79031811737%26prv_name=TEST%26ccy=RUB%26comment=test%26command=bill"
~~~

<ul class="nestedList url">
    <li><h3>URL</h3>
    </li>
</ul>

<aside class="notice">
To receive notifications, specify your notifications server URL in <b>Server notification</b> section of <b>Settings</b> section at <a href="https://kassa.qiwi.com">QIWI partners web site</a>.
<ul class="nestedList notice_image">
    <li><h3>Details</h3>
    <ul>
         <li><img src="/images/pull_rest_auth_kassa.png"/></li>
    </ul>
    <ul>
         <li><img src="/images/pull_rest_notifications_kassa.png"/></li>
    </ul>
    <ul>
        <li>Click "Create password and save" to obtain <a href="#basic_notify">password</a> for notification Basic authorization</li>
    </ul>
    </li>
</ul>
</aside>


<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic *** - for <a href="#basic_notify">login/password authorization</a></li>
             <li>X-Api-Signature: *** - for <a href="#sign_notify">digital signature authorization</a></li>
             <li>Accept: text/json</li>
             <li>Content-type: application/x-www-form-urlencoded</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Invoice parameters are in the request's body.</span>
    </li>
</ul>

Parameter|Description|Type|Required
---------|--------|---|------
bill_id |Merchant invoice number | String|Y
status | [Current invoice status](#status)|String|Y
amount | The invoice amount. The number is rounded down with two decimal places | Number(6.2)|Y
user | The QIWI Wallet user’s ID, to whom the invoice is issued. It is the user’s phone number with `tel:` prefix | String|Y
prv_name |  Merchant’s site name specified on [kassa.qiwi.com](https://kassa.qiwi.com) in "Settings" section | String|Y
ccy | Invoice currency identifier (Alpha-3 ISO 4217 code) | String(3)|Y
comment | Comment to the invoice | String(255)|Y
command | Always `bill` by default | String |Y

<aside class="notice">As new parameters of the invoice might be introduced on QIWI Wallet side, a list of invoice parameters in HTTP-request should not be fixed on the merchant's side and should be taken from the request itself.</aside>

<h3 class="request">Response ←</h3>

> Example of XML Response to notification

~~~xml
HTTP/1.1 200 OK
Content-Type: text/xml

<?xml version="1.0"?>
<result>
<result_code>0</result_code>
</result>
~~~

Response must be in XML format.

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Content-type: text/xml</li>
        </ul>
    </li>
</ul>


<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>


XML Tag|Description
--------|--------
result|Grouping tag. Describes notification processing result.
result_code|Notification result code (positive integer). We recommend that the result codes returned by the merchant be in accordance with [Notification codes table](#notify_codes).

<aside class="warning">
Any response with result code other than 0 ("Success") and/or HTTP status code other than 200 (OK) will be treated as a temporary error. Thus the server will continue repeating requests with increased time intervals within the next 24 hours (<b>50 attempts in total</b>) until it gets a response with result code 0 ("Success") and HTTP status code 200 (OK).
</aside>

<aside class="notice">
If the response with result code 0 ("Success") and HTTP status code 200 has not been received within 24 hours since the first request, QIWI Wallet server will stop sending the requests. The merchant would receive an email  with new invoice status code and indication on the possible technical issues on the merchant’s server side.
</aside>

<aside class="notice">
To receive notifications merchant must whitelist following IP subnets connected by 80, 443 ports exclusively:

<li>91.232.230.0/23</li>
<li>79.142.16.0/20</li>
</aside>

## Authorization on Merchant's Server {#notifications_auth}

Merchant's server should use [basic-authorization](#basic_notify) or [authorization by signature](#sign_notify). Merchant may also use client SSL certificate verification (self-signed cerificates may be used as well). QIWI Wallet server certificate should be verified in HTTPS requests.

<aside class="notice">
If the client SSL-certificate is self-generated and is not issued by one of the standard certification centers, this certificate should be uploaded to the QIWI Wallet server via <b>Certificate</b> field in <b>Settings - Protocols details - REST-protocol</b> section of <a href="https://kassa.qiwi.com">QIWI partners</a> web site).

<ul class="nestedList notice_image">
   <li><h3>Details</h3>
        <ul>
           <li><img src="/images/pull_rest_notifications_cert_kassa.png" /></li>
        </ul>
   </li>
</ul>

The merchant's certificate is treated as trusted after the upload. Certificate must be in one of the following formats:
<ul><li>PEM (text file with .pem extension) – (Privacy-enhanced Electronic Mail) BASE64 encoded DER certificate placed between <i>-----BEGIN CERTIFICATE-----</i> and <i>-----END CERTIFICATE-----</i> strings.</li>
<li>DER (binary file with .cer, .crt, .der extensions) – usually in binary DER format, though PEM certificates are also accepted with this extensions.</li></ul>

</aside>

## Basic authorization {#basic_notify}

> Example of notification with Basic auth

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
Authorization: Basic ***
Host: service.ru

command=bill&bill_id=BILL-1&status=paid&error=0&amount=1.00&user=tel%3A%2B79031811737&prv_name=Retail_Store&ccy=RUB&comment=test
~~~

The login is taken from [Shop ID](#auth_param) parameter. To obtain password, click on **Change notification password** button in **Protocols details - REST-protocol** section of [QIWI partners](https://kassa.qiwi.com) web site.

<ul class="nestedList">
    <li><h3>Details</h3>
        <ul>
             <li><img src="/images/pull_rest_notifications_cert_kassa.png" /></li>
        </ul>
    </li>
</ul>

## Authorization by signature {#sign_notify}

> Example of notification with Signature

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
X-Api-Signature: J4WNfNZd***V5mv2w=
Host: service.ru

command=bill&bill_id=LocalTest17&status=paid&error=0&amount=0.01&user=tel%3A%2B78000005122&prv_name=Test&ccy=RUB&comment=Some+Descriptor
~~~

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

<aside class="notice">
Merchant should enable "Use HMAC sign instead of basic auth" flag in <b>Protocols details - REST-protocol</b> section of <a href="https://kassa.qiwi.com">QIWI partners</a> web site.

<ul class="nestedList notice_image">
   <li><h3>Details</h3>
        <ul>
           <li><img src="/images/pull_rest_notifications_kassa.png" /></li>
        </ul>
   </li>
</ul>
</aside>

The HTTP header `X-Api-Signature` with signature is added to the POST-request. Signature is calculated as HMAC algorithm with SHA1-hash function.

* Parameters' separator is `|`.
* Signed are all the parameters in the original [invoice request](#invoice).
* Parameters are in alphabetical order and UTF-8 byte-encoded.
* Secret key for signature is the [password](#basic_notify) for notification basic-authorization.

Signature verification algorithm is as follows:

1. Prepare a string of all parameters values from the notification POST-request sorted in alphabetical order and separated by `|`:

    `{parameter1}|{parameter2}|…`

    where `{parameter1}` is the value of the notification parameter. All values should be treated as strings.

2. Transform obtained string and password for the notification [basic-authorization](#basic_notify) into bytes encoded in UTF-8.
3. Apply HMAC-SHA1 function:

    `hash = HMAС(SHA1, Notification_password_bytes, Invoice_parameters_bytes)`
    Where:

    * `Notification_password_bytes` – secret key (bytecoded notification password);
    * `Invoice_parameters_bytes` – bytecoded POST-request body;
    * `hash` – hash-function result.

4. Transform HMAC-hash value into bytes with UTF-8 and Base64-encode it.
5. Compare `X-Api-Signature` header's value with the result of step 4.

## PHP Implementation Example {#php_apisign}

The given PHP example implements notification authorization by signature verification. Open the _PHP_ tab on the right.

## Notification Codes {#notify_codes}

Code|Description
--------------|--------
0|Success
5|The format of the request parameters is incorrect
13|Database connection error
150|Incorrect password
151|Signature authorization failed
300|Server connection error
