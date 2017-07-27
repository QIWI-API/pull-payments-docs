# Pull REST API {#pull-payments-api_ru}

###### Последнее обновление: 2017-07-11 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_pull-payments-api_ru.html.md)

## Последовательность операций {#steps}

![Operation Flow](images/pullrest_1.png)

* Пользователь формирует заказ на сайте провайдера.

* Далее провайдер выполняет запрос [Cоздать счет](#invoice_rest) с параметрами авторизации.

* После успешного создания счета рекомендуется перенаправлять пользователя на [платежную форму](#checkout_ru) Visa QIWI Wallet. Иначе пользователю потребуется оплатить счет через любой другой интерфейс Visa QIWI Wallet (сайт qiwi.com, платежный терминал QIWI, мобильное приложение Android или iOS).

* Если провайдер включил отправку [уведомлений на сервер провайдера](#notification_ru), то после проведения платежа система Visa QIWI Wallet высылает уведомление на сервер провайдера об оплате данного счета, либо, если пользователь отклонил счет, о неоплате. Уведомления об оплате счета содержат параметры авторизации, которые необходимо проверять на сервере провайдера.

* Провайдер может:
  * [запросить текущий статус оплаты счета](#invoice-status),
   * [отменить счет](#cancel) (при условии, что он еще не был оплачен).

* После подтверждения оплаты счета провайдер исполняет заказ пользователя.

## Авторизация {#auth_rest_api}

Запросы мерчанта авторизуются посредством HTTP basic-авторизации. Для авторизации используются [API ID и API password](#auth_param).<br>


~~~shell
user@server:~$ curl "адрес сервера"
  --header "Authorization: Basic MjMyNDQxMjM6NDUzRmRnZDQ0Mw=="
~~~

<aside class="notice">
Заголовок представляет собой параметр Authorization, значение которого представлено как: Basic Base64(API_ID:API_PASSWORD)
</aside>


## Выставление счета за покупку {#invoice_rest}


Запрос выставляет новый счет на указанный номер телефона (номер кошелька QIWI Wallet). Тип запроса - HTTP PUT.

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/test234578"
  -X PUT --header "Accept: text/json"
  --header "Authorization: Basic ***"
  -d 'user=tel%3A%2B79161111111&amount=1.00&ccy=RUB&comment=uud_TEST7&lifetime=2016-09-25T15:00:00'


HTTP/1.1 200 OK
Content-Type: text/json
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
        "comment":
        "test"
     }
  }
}
~~~

~~~http
PUT /api/v2/prv/2042/bills/BILL-1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

user=tel%3A%2B79031234567%26amount=10.0%26ccy=RUB%26comment=test%26lifetime=2012-11-25T09%3A00%3A00


HTTP/1.1 200 OK
Content-Type: text/json
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
        "comment":
        "test"
     }
  }
}
~~~

~~~php
<?php
//Идентификатор магазина из вкладки "Данные магазина"
//https://ishop.qiwi.com/options/http.action
$SHOP_ID = "21379721";
//API ID из вкладки "Данные магазина"
//https://ishop.qiwi.com/options/rest.action
$REST_ID = "62573819";
//API пароль из вкладки "Данные магазина"
//https://ishop.qiwi.com/options/rest.action
$PWD = "**********";
//ID счета
$BILL_ID = "99111-ABCD-1-2-1";
$PHONE = "79191234567";

$data = array(
    "user" => "tel:+" . $PHONE,
    "amount" => "1000.00",
    "ccy" => "RUB",
    "comment" => "Все очень хорошо",
    "lifetime" => "2015-01-30T15:35:00",
    "pay_source" => "qw",
    "prv_name" => "Хороший магазин"
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
//Необязательный редирект пользователя
$url = 'https://bill.qiwi.com/order/external/main.action?shop='.$SHOP_ID.'&
transaction='.$BILL_ID.'&successUrl=http%3A%2F%2Fieast.ru%2Findex.php%3Froute%3D
payment%2Fqiwi%2Fsuccess&failUrl=http%3A%2F%2Fieast.ru%2Findex.php%3Froute%3D
payment%2Fqiwi%2Ffail&pay_source=card';
echo '<br><br><b><a href="'.$url.'">Ссылка переадресации для оплаты счета</a></b>';
?>
~~~
<h3 class="request method">Запрос → PUT</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>В pathname PUT-запроса используются два параметра счета:</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор проекта, который отображается в пункте "ID проекта" на партнерском сайте ishop.qiwi.com)</li>
             <li><strong>bill_id</strong> - уникальный идентификатор счета в системе провайдера.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
             <li>Accept: text/xml или Accept: application/xml - формат ответа XML</li>
             <li>Content-type: application/x-www-form-urlencoded; charset=utf-8</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса как formdata</span>
    </li>
</ul>

Параметр|Описание|Тип|Обяз.
---------|--------|---|------
user | Идентификатор номера QIWI Wallet, на который выставляется счет (в международном формате), с префиксом "tel:" | String(20)|+
amount | Сумма, на которую выставляется счет. Способ округления зависит от валюты | Number(6.3)|+
ccy | Идентификатор валюты (Alpha-3 ISO 4217 код). Может использоваться любая валюта, предусмотренная договором с КИВИ | String(3)|+
comment | Комментарий к счету | String(255)|+
lifetime | Дата, до которой счет будет доступен для оплаты. Если счет не будет оплачен до этой даты, ему присваивается финальный статус и последующая оплата станет невозможна.<br> **Внимание! По истечении 28 суток от даты выставления счет автоматически будет переведен в финальный статус.**|dateTime|+
pay_source |<br>"mobile" - оплата счета будет производиться с баланса мобильного телефона пользователя, <br>"qw" – любым способом через интерфейс Visa QIWI Wallet.<br> По умолчанию "qw" |String|-
prv_name|Название провайдера.| String(100)|-


<h3 class="request">Ответ ←</h3>

Формат ответа зависит от заголовка "Accept" в исходном запросе:

* Accept: text/json или Accept: application/json - формат ответа JSON
* Accept: text/xml или Accept: application/xml - формат ответа XML


[Параметры ответа](#responses_ru)

[Ответ в случае ошибки](#response_error)

## Проверка статуса оплаты счета {#invoice-status}

Запрос позволяет проверить текущий статус оплаты счета клиентом.

<h3 class="request method">Запрос → GET</h3>

~~~http
GET /api/v2/prv/2042/bills/BILL-1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

HTTP/1.1 200 OK
Content-Type: text/json
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
        "comment": "test"
     }
  }
}
~~~

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/sdf23452435"
  --header "Authorization: Basic ***"
  --header "Accept: text/json"

HTTP/1.1 200 OK
Content-Type: text/json
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
        "comment": "test"
     }
  }
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Параметры передаются в pathname.</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор проекта, который отображается в пункте ID проекта на партнерском сайте ishop.qiwi.com)</li>
             <li><strong>bill_id</strong> - уникальный идентификатор счета в системе провайдера.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic ***</li>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
             <li>Accept: text/xml или Accept: application/xml - формат ответа XML</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

Формат ответа зависит от заголовка Accept в исходном запросе:

* Accept: text/json или Accept: application/json - формат ответа JSON
* Accept: text/xml или Accept: application/xml - формат ответа XML

[Параметры ответа](#responses_ru)

[Ответ в случае ошибки](#response_error)



## Отмена неоплаченного счета {#cancel}

Запрос позволяет отменить неоплаченный клиентом счет.

<h3 class="request method">Запрос → PATCH</h3>

~~~http
PATCH /api/v2/prv/2042/bills/BILL-1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

status=rejected

HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "bill": {
         "bill_id": "BILL-2",
         "amount": "10.00",
         "ccy": "RUB",
         "status": "rejected",
         "error": 0,
         "user": "tel:+79031234567",
         "comment": "test"
      }
   }
}
~~~

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/sdf23452435"
  -X PATCH
  --header "Authorization: Basic ***"
  --header "Accept: text/json"
  --header "Content-type: application/x-www-form-urlencoded; charset=utf-8"
  -d 'status=rejected'


HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "bill": {
         "bill_id": "BILL-2",
         "amount": "10.00",
         "ccy": "RUB",
         "status": "rejected",
         "error": 0,
         "user": "tel:+79031234567",
         "comment": "test"
      }
   }
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Параметры передаются в pathname.</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор проекта, который отображается в пункте ID проекта на партнерском сайте ishop.qiwi.com)</li>
             <li><strong>bill_id</strong> - уникальный идентификатор счета в системе провайдера.</li>
         </ul>
         <ul>
         <strong>Параметр передается в body запроса.</strong>
             <li><strong>status</strong> - строка "rejected" (статус для отмены).</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic ***</li>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
             <li>Accept: text/xml или Accept: application/xml - формат ответа XML</li>
             <li>Content-type: application/x-www-form-urlencoded; charset=utf-8</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

Формат ответа зависит от заголовка "Accept" в исходном запросе:

* Accept: text/json или Accept: application/json - формат ответа JSON
* Accept: text/xml или Accept: application/xml - формат ответа XML

[Параметры ответа](#responses_ru)

[Ответ в случае ошибки](#response_error)



## Возврат оплаченного счета {#refund}

С помощью данного запроса можно произвести полный или частичный возврат средств по счету, оплаченному клиентом, на его учетную запись Visa QIWI Wallet. При этом создается платеж, обратный платежу на оплату счета. Валюта платежа совпадает с валютой исходного счета.

По одному и тому же счету можно выполнять несколько операций возврата, при условии что:

* сумма всех операций возврата не превышает суммы исходного счета;
* для разных операций возврата одного счета используются разные идентификаторы.

<aside class="warning">
Если сумма, переданная в запросе, превышает сумму самого счета либо сумму счета, оставшуюся после предыдущих возвратов, в ответе будет возвращен код ошибки 242.
</aside>

### Последовательность операций

![Refund Operation Flow](images/pullrest_2.png)

* Провайдер отправляет запрос на осуществление возврата.
* Чтобы убедиться, что возврат платежа проведен успешно, можно периодически опрашивать сервис Visa QIWI Wallet о [текущем статусе возврата](#refund_status) до получения финального статуса.
* Данный сценарий можно повторять несколько раз до тех пор, пока счет не будет полностью отменен (возвращена вся сумма).

<h3 class="request method">Запрос → PUT</h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/test234578/refund/br1"
  -v -w "%{http_code}"
  -X PUT
  --header "Accept: text/json"
  --header "Authorization: Basic ***"
  --header "Content-type: application/x-www-form-urlencoded; charset=utf-8"
  -d 'amount=5.0'

HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": "br1",
         "amount": "5.00",
         "status": "success",
         "error": 0
      }
   }
}
~~~

~~~http
PUT /api/v2/prv/2042/bills/BILL-1/refund/br1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

amount=5.0

HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": "br1",
         "amount": "5.00",
         "status": "success",
         "error": 0
      }
   }
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a>/refund/<a>refund_id</a></span></h3>
        <ul>
        <strong>Параметры передаются в pathname.</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор проекта, который отображается в пункте ID проекта на партнерском сайте ishop.qiwi.com)</li>
             <li><strong>bill_id</strong> - уникальный идентификатор счета в системе провайдера.</li>
             <li><strong>refund_id</strong> - идентификатор операции, уникальный в рамках операций возврата счета <br> bill_id. Формат идентификатора: строка от 1 до 9 символов, содержащая только прописные или строчные латинские буквы (a-z, A-Z) и цифры от 0 до 9.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic ***</li>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
             <li>Accept: text/xml или Accept: application/xml - формат ответа XML</li>
             <li>Content-Type: application/x-www-form-urlencoded; charset=utf-8</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса как formdata</span>
    </li>
</ul>

Параметр|Описание|Тип
---------|--------|---|------
amount | Сумма возврата. Должна быть меньше либо равна сумме счета `{bill_id}`, по которому производится возврат. Способ округления зависит от валюты счета | Number(6.3)


<h3 class="request">Ответ ←</h3>

Формат ответа зависит от заголовка "Accept" в исходном запросе:

* Accept: text/json или Accept: application/json - формат ответа JSON
* Accept: text/xml или Accept: application/xml - формат ответа XML

[Параметры ответа](#response_refund)

[Ответ в случае ошибки](#response_error)

## Проверка статуса возврата {#refund_status}

С помощью данного запроса можно проверить текущий статус операции возврата средств по счету.

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/test234578/refund/br1"
  -v -w "%{http_code}"
  --header "Accept: text/json"
  --header "Authorization: Basic ***"


HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": "br1",
         "amount": "5.00",
         "status": "success",
         "error": 0
      }
   }
}
~~~

~~~http
GET /api/v2/prv/2042/bills/BILL-1/refund/br1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com


HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": "br1",
         "amount": "5.00",
         "status": "success",
         "error": 0
      }
   }
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a>/refund/<a>refund_id</a></span></h3>
        <ul>
        <strong>Параметры передаются в pathname.</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор проекта, который отображается в пункте ID проекта на партнерском сайте ishop.qiwi.com)</li>
             <li><strong>bill_id</strong> - уникальный идентификатор счета в системе провайдера.</li>
             <li><strong>refund_id</strong> - идентификатор операции, уникальный в рамках операций возврата счета bill_id. Формат идентификатора: строка от 1 до 9 символов, содержащая только прописные или строчные латинские буквы (a-z, A-Z) и цифры от 0 до 9</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic ***</li>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
             <li>Accept: text/xml или Accept: application/xml - формат ответа XML</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

Формат ответа зависит от заголовка "Accept" в исходном запросе:

* Accept: text/json или Accept: application/json - формат ответа JSON
* Accept: text/xml или Accept: application/xml - формат ответа XML


[Параметры ответа](#response_refund)

[Ответ в случае ошибки](#response_error)
