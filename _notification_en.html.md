# Notifications {#notification_en}

###### Last update: 2017-07-11 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_notification_en.html.md)

Notifications are POST-requests (callbacks) from Visa QIWI Wallet server containing all relevant data of the invoice serialized as HTTP-request parameters (encoded by UTF-8) plus parameter `command=bill`.

<h3 class="request method">Request → POST</h3>

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
Authorization: Basic ***

bill_id=BILL-1&status=paid&error=0&amount=1.00&user=tel%3A%2B79031811737&prv_name=Retail_Store&ccy=RUB&comment=test&command=bill
~~~

To receive notifications (callbacks on invoice status changing), use **Turn on notifications** flag in **Settings - Protocols details - REST-protocol** section of [QIWI](ishop.qiwi.com) web site. To receive these notifications merchant’s server should accept specific HTTP-requests on ports 80, 443.

The notification data are transmitted as `application/x-www-form-urlencoded` content type encoded by UTF-8. Parameters, transmitted in the request body, are URL-encoded. 

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic ***</li>
             <li>Accept: text/json</li>
             <li>Content-type: application/x-www-form-urlencoded</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response → POST</h3>

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
