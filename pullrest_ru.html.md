---
title: QIWI Pull REST API 2.1

search: true

metatitle: QIWI Pull REST API 2.1

metadescription: QIWI Wallet Pull API открывает доступ к операциям со счетами в Visa QIWI Wallet из вашего приложения. Поддерживаются операции выставления и отмены счетов, возврата средств по счетам, а также проверки статуса выполнения операций.

language_tabs:
  - shell
  - php: PHP
  - http: HTTP
  - xml: XML 
  - json: JSON

services:
 - <a href='#'>Swagger</a>  |  <a href='#'>Qiwi Demo</a>


toc_footers:
 - <a href='/'>На главную</a>
 - <a href='http://pullapi-test.qiwi.com'>Песочница</a>
---

# Введение

QIWI Wallet Pull API открывает доступ к операциям со счетами в Visa QIWI Wallet из вашего приложения. Поддерживаются операции выставления и отмены счетов, возврата средств по счетам, а также проверки статуса выполнения операций.

## Способы оплаты

* Пользователи могут оплачивать счета Visa QIWI Wallet
  * в интерфейсах платежной формы QIWI (bill.qiwi.com)
  * на сайте [qiwi.com](#https://qiwi.com)
  * в мобильных приложений QIWI
    * с баланса своего Visa QIWI Wallet
    * с баланса мобильного телефона или с любой карты Visa/MasterCard.
  * также доступна оплата счетов наличными в QIWI Терминалах.

**Данное API можно использовать только после регистрации и подключения к https://ishop.qiwi.com.**

## Способы интеграции

Для работы с QIWI Wallet Pull API доступны следующие способы:

* [Pull REST API](#pull_rest_api) - полнофункциональное API для всех операций со счетами.

* [Веб-форма выставления счета](#http) - Вызов веб-формы авторизуется с помощью API ID и подписи запроса, либо выполняется без авторизации. Не требует сложной реализации, но ограничена по функциональности - поддерживает только выставление счета.

## Служебные данные {#auth_param}

<ul class="nestedList params">
    <li><h3>Авторизация и работа с формами</h3><span>Данные могут быть получены на сайте <strong>ishop.qiwi.com</strong></span>
    </li>
</ul>

`Параметр|Описание|Тип|Обяз.
 ---------|--------|---|------
 API_ID | Идентификатор для авторизации провайдера в API | Integer| +
 API_PASSWORD | Пароль для авторизации в API| String | +
 ID проекта | числовой идентификатор провайдера (идентификатор проекта или PRV_ID) | Integer | +



<aside class="notice">
Получить служебне данные можно на партнерском сайте <a href='http://ishop.qiwi.com'>ishop.qiwi.com</a> в разделе "Протоколы - REST-протокол - Аутентификационные данные".

<ul class="nestedList notice_image">
    <li><h3>Подробнее</h3>
        <ul>
             <li><img src="images/pull_rest_auth.png" /></li>
        </ul>
    </li>
</ul>

</aside>



# Qiwi Wallet Pull REST API {#pull_rest_api}

### Последовательность операций

<img src="images/pullrest_1.png" />

* Пользователь формирует заказ на сайте провайдера.

* Далее провайдер выполняет запрос [Cоздать счет](#invoice_rest) с параметрами авторизации.

* После успешного создания счета рекомендуется перенаправлять пользователя на [платежную форму](#checkout) Visa QIWI Wallet. Иначе пользователю потребуется оплатить счет через любой другой интерфейс Visa QIWI Wallet (сайт qiwi.com, платежный терминал QIWI, мобильное приложение Android или iOS).

* Если провайдер включил отправку [уведомлений на сервер провайдера](#notification), то после проведения платежа система Visa QIWI Wallet высылает уведомление на сервер провайдера об оплате данного счета, либо, если пользователь отклонил счет, о неоплате. Уведомления об оплате счета содержат параметры авторизации, которые необходимо проверять на сервере провайдера.

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

<h3 class="request method put">Запрос → </h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>В pathname PUT-запроса используется два параметра счета:</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор проекта, который отображается в пункте ID проекта на партнерском сайте ishop.qiwi.com)</li>
             <li><strong>bill_id</strong> - уникальный идентификатор счета в системе провайдера.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json</li>
             <li>Content-type: application/x-www-form-urlencoded; charset=utf-8</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Параметры передаются в теле запроса как formdata</span>
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



[Параметры ответа](#response_bill)

[Ответ в случае ошибки](#response_error)

## Проверка статуса оплаты счета {#invoice-status}
позволяет проверить текущий статус оплаты счета клиентом.

<h3 class="request method get">Запрос → </h3>

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
             <li>Accept: text/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

[Параметры ответа](#response_bill)

[Ответ в случае ошибки](#response_error)



## Отмена неоплаченного счета {#cancel}
позволяет отменить неоплаченный клиентом счет.

<h3 class="request method patch">Запрос → </h3>

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
             <li><strong>status</strong> - строка "rejected" (статус для отмены), передается в body запроса.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic ***</li>
             <li>Accept: text/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

[Параметры ответа](#response_bill)

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

<img src="images/pullrest_2.png" />

* Провайдер отправляет запрос на осуществление возврата.
* Чтобы убедиться, что возврат платежа проведен успешно, можно периодически опрашивать сервис Visa QIWI Wallet о [текущем статусе возврата](#refund_status) до получения финального статуса.
* Данный сценарий можно повторять несколько раз до тех пор, пока счет не будет полностью отменен (возвращена вся сумма).

<h3 class="request method put">Запрос → </h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/test234578/refund/122swbill"
  -v -w "%{http_code}" 
  -X PUT 
  --header "Accept: text/json" 
  --header "Authorization: Basic ***"  
  -d 'amount=10.0'

HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": 1,
         "amount": "5.00",
         "status": "success",
         "error": 0
      }
   }
}
~~~

~~~http
PUT /api/v2/prv/2042/bills/BILL-1/refund/122swbill HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

amount=10.0

HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": 1,
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
             <li><strong>refund_id</strong> - идентификатор операции, уникальный в рамках операций возврата счета <br> `{bill_id}`. Формат идентификатора: строка от 1 до 9 символов, содержащая только прописные или строчные латинские буквы (a-z, A-Z) и цифры от 0 до 9.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic ***</li>
             <li>Accept: text/json</li>
             <li>Content-Type: application/x-www-form-urlencoded; charset=utf-8</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Параметры передаются в теле запроса как formdata</span>
    </li>
</ul>

Параметр|Описание|Тип
---------|--------|---|------
amount | Сумма возврата. Должна быть меньше либо равна сумме счета `{bill_id}`, по которому производится возврат. Способ округления зависит от валюты счета | Number(6.3)


<h3 class="request">Ответ ←</h3>

[Параметры ответа](#response_refund)

[Ответ в случае ошибки](#response_error)

## Проверка статуса возврата {#refund_status}

С помощью данного запроса можно проверить текущий статус операции возврата средств по счету.

<h3 class="request method get">Запрос → </h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/test234578/refund/122swbill"
  -v -w "%{http_code}" 
  --header "Accept: text/json" 
  --header "Authorization: Basic ***"


HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": 1,
         "amount": "5.00",
         "status": "success",
         "error": 0
      }
   }
}
~~~

~~~http
GET /api/v2/prv/2042/bills/BILL-1/refund/122swbill HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8


HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": 1,
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
             <li><strong>refund</strong> - идентификатор операции, уникальный в рамках операций возврата счета `bill_id`. Формат идентификатора: строка от 1 до 9 символов, содержащая только прописные или строчные латинские буквы (a-z, A-Z) и цифры от 0 до 9</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic ***</li>
             <li>Accept: text/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

[Параметры ответа](#response_refund)

[Ответ в случае ошибки](#response_error)

# Форма выставления счета {#http}


Клиенту отображается платежная форма с выбором способа оплаты выставленного счета.

* Вызов веб-формы может выполняться двумя способами:

  * WF - вызов формы без авторизации. Номер телефона, на который будет выставлен счет, должен быть указан в параметрах вызова.

  * WF w/auth - в параметрах вызова формы содержатся идентификатор для авторизации и цифровая подпись. Номер телефона может указать клиент непосредственно на веб-форме. Данный способ находится в статусе бета-версии. Для его использования необходимо обратиться в Техническую поддержку КИВИ (bss@qiwi.ru).

### Последовательность операций:

1. Пользователь формирует заказ на сайте провайдера.
2. Далее провайдер выполняет вызов веб-формы. В запросе может использоваться авторизация провайдера. В способе WF w/auth при отсутствии номера телефона в параметрах вызова пользователь указывает свой номер телефона на форме.
3. В случае успешного создания счета пользователь автоматически переходит на [платежную форму](#checkout) Visa QIWI Wallet. 
4. Если провайдер включил отправку [уведомлений на сервер провайдера](#notification), то после проведения платежа система Visa QIWI Wallet высылает уведомление на сервер провайдера об оплате данного счета, либо, если пользователь отклонил счет, о неоплате. Уведомления об оплате счета содержат параметры авторизации, которые необходимо проверять на сервере провайдера.
5. После подтверждения оплаты счета провайдер исполняет заказ пользователя.

<h3 class="request method redirect">Запрос → </h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://bill.qiwi.com/order/external/create.action</span></h3></li>
</ul>

~~~http
  Без авторизации
GET /order/external/create.action?comm=test&txn_id=0000&from=000000&summ=1.11&successUrl=http%3A%2F%2Ftest.ru%3Fcurrency=643&to=%2B71234567890 HTTP/1.1
Host: bill.qiwi.com
~~~

<ul class="nestedList example">
    <li><h3>Пример без авторизации</h3><span><a>https://bill.qiwi.com/order/external/create.action?comm=test&txn_id=0000&from=000000&summ=1.11&successUrl=http%3A%2F%2Ftest.ru%3Fcurrency=643&to=%2B71234567890</a></span>
    </li>
</ul>

~~~http
  C авторизацией
GET /order/external/create.action?from=260831&txn_id=q115928&summ=1.12&api_id=46835183&currency=RUB&sign=7d3e8b8df7c8ed70b1089c6a5ae86eddd1ee HTTP/1.1
Host: bill.qiwi.com
~~~

<ul class="nestedList example">
    <li><h3>Пример c авторизацией</h3><span><a>https://bill.qiwi.com/order/external/create.action?from=260831&txn_id=q115928&summ=1.12&api_id=46835183&currency=RUB&sign=7d3e8b8df7c8ed70b1089c6a5ae86eddd1ee</a></span>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>В ссылке на веб-форму указываются параметры счета.</span></li>
</ul>





Параметр|Описание|Тип|WF w/auth|WF|Обяз.
---------|--------|---|---------|---|----
from | Идентификатор провайдера. Идентификатор указан в настройках HTTP-протокола в личном кабинете провайдера на сайте ishop.qiwi.com|Integer|+|+|+
to | Идентификатор номера QIWI Wallet, на который выставляется счет (в международном формате). Не обязателен для WF w/auth: если не указан, то пользователю отображается веб-форма с полем ввода номера телефона и счет выставляется только после заполнения номера | String(20)|+|+|WF
summ | Сумма, на которую выставляется счет. Способ округления зависит от валюты | Number(6.3)|+|+|+
currency | Идентификатор валюты (Alpha-3 ISO 4217 код). Может использоваться любая валюта, предусмотренная договором с КИВИ | String(3)|+|+|+
txn_id|Уникальный идентификатор счета в системе провайдера|String(30)|+|+|WF w/auth
api_id|[Параметры авторизации](#auth_param)|Integer|+|-|WF w/auth
sign|[Подпись запроса](#http_sign)|String|+|-|WF w/auth
comm | Комментарий к счету. Если не указан для WF w/auth, и не указан параметр `to`, то пользователю отображается веб-форма с полями ввода номера телефона и комментария. Счет выставляется только после заполнения номера| String(255)|+|+|-
lifetime | Дата, до которой счет будет доступен для оплаты. Если счет не будет оплачен до этой даты, ему присваивается финальный статус и последующая оплата станет невозможна.<br> **Внимание! По истечении 28 суток от даты выставления счет автоматически будет переведен в финальный статус.**|ГГГГ-ММ-ДДTЧЧММ|+|+|-
iframe| Признак отображения страницы в iframe (более компактный вид, удобный для встраивания ее в сайт провайдера).|Логический, `true`/`false`<br>По умолчанию `false`|+|+|-
successUrl|URL для переадресации в случае успешного создания транзакции в Visa Qiwi Wallet. Ссылка должна вести на сайт провайдера. Если пользователь выбрал на платежной форме способ оплаты, отличный от оплаты с баланса Visa QIWI Wallet, то переадресация на сайт провайдера не выполняется.|URL-закодированная строка|+|+|-
failUrl|URL для переадресации в случае неуспеха при создании транзакции в Visa Qiwi Wallet. Ссылка должна вести на сайт провайдера. Если пользователь выбрал на платежной форме способ оплаты, отличный от оплаты с баланса Visa QIWI Wallet, то переадресация на сайт провайдера не выполняется.|URL-закодированная строка|+|+|-
target|Флаг, показывающий, что ссылки в параметрах <br>`successUrl` / `failUrl` открываются в iframe. Если отсутствует, то считается выключенным|Строка (только `iframe`)|+|+|-
pay_source |Способ оплаты по умолчанию, который необходимо отобразить пользователю при открытии платежной формы. Возможные значения:<br>`qw` – оплата с баланса Visa QIWI Wallet;<br> `mobile` – оплата с баланса мобильного телефона;<br> `card` – оплата банковской картой;<br> `wm` – оплата с привязанного кошелька WebMoney;<br> `ssk` – оплата наличными в терминале QIWI.<br>Если способ оплаты не доступен, пользователю отображается предупреждение, при этом на странице можно выбрать другие способы оплаты. |String|+|+|-  

## Подпись {#http_sign}

Запрос выставления счета с авторизацией требует вычисления параметра `sign` (цифровой подписи). 

<aside class="notice">
Для вычисления подписи используется API password – пароль, соответствующий <a href='#auth_param'>API_ID</a>
</aside>

Алгоритм вычисления подписи:

1.	Получить строку, содержащую значения всех обязательных параметров GET-запроса и параметра `lifetime` (если он присутствует в запросе) в алфавитном порядке перечисления параметров, разделенных символами `|`: 

    `Invoice_parameters = "{api_id}|{currency}|{from}|{lifetime}|{summ}|{txn_id}"`

    где `{parameter}` – значение соответствующего параметра запроса.

2.	Вычислить HMAC-хэш с шифрованием SHA256 от полученной строки и ключом, равным API password (пароль к API ID):
 
    `sign = HMAС(SHA256, API_password, Invoice_parameters)`

Пример генерации подписи на вкладке **PHP** справа.

~~~php
<?php

?>
~~~

# Форма оплаты {#checkout}

Провайдер может предложить пользователю немедленно оплатить счет с помощью переадресации на платежную форму посредством HTTP GET-запроса по адресу:

<h3 class="request method redirect">Запрос → </h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://bill.qiwi.com/order/external/main.action</span></h3>
    </li>
</ul>

<aside class="notice">
В ответ сервер формирует на сайте Visa QIWI Wallet страницу с выставленным счетом и выбором способа оплаты счета.

Если провайдер использует выставление счета через [веб-форму](#http), данное действие выполняется автоматически.
</aside>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Параметр|Тип|Описание|Обяз.
---------|--------|-------|-----
shop| Строка|Идентификатор провайдера. Соответствует параметру `{prv_id}` из [запроса на выставление счета](#invoice-rest).|+
transaction|Строка|Идентификатор счета в информационной системе провайдера. Соответствует параметру `{bill_id}` из запроса на выставление счета.|+
iframe|Логический, true/false| Признак отображения страницы в iframe (более компактный вид, удобный для встраивания ее в сайт провайдера). По умолчанию `false`|-
successUrl|URL-закодированная строка|URL для переадресации в случае успешного создания транзакции в Visa Qiwi Wallet. Ссылка должна вести на сайт провайдера. Если пользователь выбрал на платежной форме способ оплаты, отличный от оплаты с баланса Visa QIWI Wallet, то переадресация на сайт провайдера не выполняется.|-
failUrl|URL-закодированная строка|URL для переадресации в случае неуспеха при создании транзакции в Visa Qiwi Wallet. Ссылка должна вести на сайт провайдера. Если пользователь выбрал на платежной форме способ оплаты, отличный от оплаты с баланса Visa QIWI Wallet, то переадресация на сайт провайдера не выполняется.|-
target|Строка "iframe"|Флаг, показывающий, что ссылки в параметрах <br>`successUrl` / `failUrl` открываются в iframe. Если отсутствует, то считается выключенным|-
pay_source|Строка| Способ оплаты по умолчанию, который необходимо отобразить пользователю при открытии платежной формы. Возможные значения:<br>`qw` – оплата с баланса Visa QIWI Wallet;<br> `mobile` – оплата с баланса мобильного телефона;<br> `card` – оплата банковской картой;<br> `wm` – оплата с привязанного кошелька WebMoney;<br> `ssk` – оплата наличными в терминале QIWI.<br>Если способ оплаты не доступен, пользователю отображается предупреждение, при этом на странице можно выбрать другие способы оплаты.|-

## Возврат на сайт провайдера

<aside class="notice">
Если в ссылке на платежную форму указан параметр `successUrl` или `failUrl`, то сайт Visa QIWI Wallet переадресует пользователя на соответствующий URL после завершения процесса оплаты.
</aside>

<aside class="warning">
Сам по себе факт перенаправления на адрес, указанный в параметре `successUrl`, не означает, что счет успешно оплачен.
Для принятия окончательного решения о предоставлении клиенту услуги или товара провайдеру необходимо дождаться [уведомления](#notification) от сервера Visa QIWI Wallet с финальным статусом счета. Если провайдер не использует уведомления, необходимо запрашивать статус счета [отдельным запросом API](#invoice-status).
</aside>

<aside class="notice">
При переадресации на сайт провайдера добавляется дополнительный параметр `order`, в котором будет передан идентификатор счета (значение параметра `transaction` из первоначального GET-запроса платежной формы). Используя этот параметр, провайдер может отобразить необходимую информацию на своей стороне.
</aside>

## Пример использования

* Провайдер после выставления счета переадресует пользователя на URL:

    * `https://bill.qiwi.com/order/external/main.action?shop=2042&transaction=1234567&successUrl=http%3A%2F%2Fmystore.com%2Fsuccess%3Fa%3D1%26b%3D2&failUrl=http%3A%2F%2Fmystore.com%2Ffail%3Fa%3D1%26b%3D2&iframe=true&target=iframe&pay_source=qw`


* Клиент видит на странице метод оплаты с баланса Visa QIWI Wallet (отображается в iframe) и оплачивает счет этим методом.

* После и успешного создания транзакции сайт Visa QIWI Wallet выполняет возврат клиента на страницу:

    * `http://mystore.com/success?a=1&b=2&order=1234567` (отображается в iframe).


* В случае неуспеха при создании транзакции сайт Visa QIWI Wallet выполняет возврат клиента на страницу:
   
    * `http://mystore.com/fail?a=1&b=2&order=1234567` (отображается в iframe).


# Уведомления об оплате счетов {#notification}

Уведомление представляет собой POST-запрос. Тело запроса содержит сериализованные данные счета в теле запроса (кодировка UTF-8), и параметр `command=bill`.

<h3 class="request method post">Запрос → </h3>

~~~shell
Пример
user@server:~$ curl "https:///qiwi-notify.php"
  -v -w "%{http_code}"
  -X POST
  --header "Accept: text/xml"
  --header "Content-Type: application/x-www-form-urlencoded; charset=utf-8"
  --Authorization: "Basic MjA0Mjp0ZXN0Cg=="
  -d 'bill_id=BILL- 1&status=paid&error=0&amount=1.00&user=tel%3A%2B79031811737&prv_nam e=TEST&ccy=RUB&comment=test&command=bill'
~~~

~~~shell
HTTP/1.1 200 OK
Content-Type: text/xml
<?xml version="1.0"?> <result><result_code>0</result_code></result>
~~~

~~~http
Пример
POST /qiwi-notify.php HTTP/1.1
Accept: application/xml
Content-type: application/x-www-form-urlencoded
Authorization: Basic MjA0Mjp0ZXN0Cg==

bill_id=BILL- 1&status=paid&error=0&amount=1.00&user=tel%3A%2B79031811737&prv_nam e=TEST&ccy=RUB&comment=test&command=bill
~~~

~~~http
HTTP/1.1 200 OK
Content-Type: text/xml
<?xml version="1.0"?> <result><result_code>0</result_code></result>
~~~

<ul class="nestedList url">
    <li><h3>URL</h3>
    </li>
</ul>

<aside class="notice">
Aдрес вашего сервера для уведомлений вы можете настроить на сайте ishop.qiwi.com в разделе "Настройки"->"REST-Протокол"->"Настройки Pull (REST) протокола"

<ul class="nestedList notice_image">
   <li><h3>Подробнее</h3>
        <ul>
           <li><img src="images/pull_rest_notification_url.png" /></li>
        </ul>
   </li>
</ul>
</aside>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic ***</li>
             <li>Accept: text/json</li>
             <li>Content-type: application/x-www-form-urlencoded</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>В ссылке на веб-форму указываются параметры счета.</span>
    </li>
</ul>

Параметр|Описание|Тип|Обяз.
---------|--------|---|------
status | [Статусы](#status)|String|+
error | [Ошибки](#errors)| String(20)|+|
amount | Сумма, на которую выставлялся счет. Способ округления зависит от валюты | Number(6.3)|+
user | Номер Visa Qiwi Wallet, на который был выставлен счет | Number|+
prv_name |  Наименование проекта, указанное на сайте ishop.qiwi.com в разделе:"Настройки"->"Данные проекта"->"Короткое наименование" | Number|+
ccy | Идентификатор валюты (Alpha-3 ISO 4217 код). Может использоваться любая валюта, предусмотренная договором с КИВИ | String(3)|+
comment | Комментарий к счету | String(255)|+
commant | `bill` - всегда по умолчанию | String |+

<h3 class="request method post">Ответ → </h3>
Ответ на запрос должен быть в формате XML.

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
Authorization: Basic ***

bill_id=BILL-1&status=paid&error=0&amount=1.00&user=tel%3A%2B79031811737&prv_name=Retail_Store&ccy=RUB&comment=test&command=bill
~~~

### Заголовки
*  `Content-type: text/xml`

~~~xml
HTTP/1.1 200 OK
Content-Type: text/xml

<?xml version="1.0"?>
<result>
<result_code>0</result_code>
</result>
~~~

<aside class="warning">
В ответе должен вернуться код результата обработки уведомления. Код результата должен находиться в теге result_code, вложенном в тег result.
</aside>

<aside class="warning">
Если в ответе код результата обработки уведомления отличается от 0 и/или код состояния HTTP отличается от 200 (OK), это интерпретируется как временная ошибка провайдера. Сервер повторяет запрос с нарастающим интервалом в течение суток (**не более 50 попыток**) до получения в ответе кода результата 0 и кода состояния HTTP 200.
</aside>

<aside class="notice">
HTTP-заголовок `Content-Type` ответа должен быть равен «text/xml». В противном случае уведомление будет считаться неуспешным.
</aside>

<aside class="notice">
Рекомендуется возвращать коды результата в соответствии с таблицей <a href="#notify_codes">кодов завершения</a>.
</aside>

<aside class="notice">
Если ответ с кодом результата 0 и кодом состояния HTTP 200 так и не был получен в указанное время, повторные уведомления от сервера Visa QIWI Wallet прекращаются, и на адрес электронной почты провайдера высылается письмо с новым статусом счета и указанием на возможную техническую неисправность в работе сервиса провайдера.
</aside>

<aside class="notice">
Для получения уведомлений (notification) о смене статуса выставленного клиенту счета необходимо включить соответствующую настройку на партнерском сайте ishop.qiwi.com (раздел **Настройки Pull (REST) протокола**, пункт **Включить уведомления**) и настроить http-cервер обработки уведомлений.
<ul class="nestedList notice_image">
    <li><h3>Подробнее</h3>
        <ul>
             <li><img src="images/pull_rest_notifications.png"/></li>
        </ul>
    </li>
</ul>
</aside>


<aside class="notice">
Для получения уведомлений провайдер должен принимать HTTP-запросы из следующих подсетей исключительно по портам 80, 443:

<li> 91.232.230.0/23</li>
<li> 79.142.16.0/20</li>
</aside>



## Авторизация уведомлений

Для авторизации можно использовать [basic-авторизацию](#basic_notify) или [авторизацию подписи](#sign_notify). При запросах уведомлений на сервер провайдера также можно использовать SSL (в том числе самоподписанный сертификат). В https-запросах необходимо проверять серверный сертификат Visa QIWI Wallet.

<aside class="notice">
Если сертификат для SSL-шифрования сгенерирован самостоятельно и не является доверенным со стороны стандартных центров сертификации, этот сертификат можно загрузить в систему Visa QIWI Wallet на партнерском сайте ishop.qiwi.com в поле <b>Сертификат</b> раздела <b>Протоколы - REST-протокол</b>. После загрузки в систему Visa QIWI Wallet данный сертификат будет считаться доверенным. Сертификат должен быть в одном из следующих форматов:
<ul><li>PEM (текстовый файл с расширением .pem) – (Privacy-enhanced Electronic Mail) закодированный BASE64 сертификат DER, помещенный между строками `-----BEGIN CERTIFICATE-----` и `-----END CERTIFICATE-----`.</li>
<li>DER (файл с расширением.cer, .crt, .der) – обычно в бинарном формате DER, однако PEM сертификаты также допускаются с таким расширением.</li></ul>
</aside>

### Basic-авторизация {#basic_notify}

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
Authorization: Basic ***

command=bill&bill_id=BILL-1&status=paid&error=0&amount=1.00&user=tel%3A%2B79031811737&prv_name=Retail_Store&ccy=RUB&comment=test
~~~

Логин равен <a href="#auth_param">ID магазина.</a> Пароль для basic-авторизации уведомления генерируется автоматически в личном кабинете мерчанта на http://ishop.qiwi.com в разделе "Настройки"->"REST-Протокол".

<ul class="nestedList notice_image">
    <li><h3>Подробнее</h3>
        <ul>
             <li><img src="images/pull_rest_notifications_pass.png" /></li>
        </ul>
    </li>
</ul>



### Авторизация подписи {#sign_notify}

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
X-Api-Signature: J4WNfNZd***V5mv2w=

command=bill&bill_id=LocalTest17&status=paid&error=0&amount=0.01&user=tel%3A%2B78000005122&prv_name=Test&ccy=RUB&comment=Some+Descriptor
~~~

Подпись уведомления отправляется в заголовке `X-Api-Signature`. Для формирования подписи используется механизм проверки целостности HMAC с хэш-функцией SHA1.

* В качестве разделителей параметров используется символ `|`.
* В подписи участвуют все параметры, которые присутствуют в исходном [запросе выставления счета](#invoice_rest).
* Параметры для подписи переводятся в байт-представление с UTF-8 и располагаются в алфавитном порядке.
* Ключ подписи равен [паролю](#basic_notify) для basic-авторизации уведомления.

Алгоритм проверки подписи:

1. Получить строку, содержащую значения всех параметров POST-запроса в алфавитном порядке перечисления параметров, разделенных символами `|`: 

   `{parameter1}|{parameter2}|…`

   где `{parameter1}` – значение параметра уведомления. Все значения при проверке подписи должны трактоваться как строки.

2. Cтроку и пароль для basic-авторизации уведомления преобразовать в байты с UTF-8.
3. Вычислить HMAC-хэш c шифрованием SHA1:

   `hash = HMAС(SHA1, Notification_password_bytes, Invoice_parameters_bytes)`
   Где:

   * `Notification_password_bytes` – ключ функции (байт-представление basic-пароля для уведомлений);
   * `Invoice_parameters_bytes` – байт-представление тела POST-запроса;
   * `hash` – результат хэш-функции.

4. HMAC-хэш преобразовать из строк в байты с использованием кодировки UTF-8 и base64-преобразовать.
5. Сравнить значение заголовка X-Api-Signature с результатом 4.

### Пример реализации - откройте вкладку *PHP* справа.

~~~php
<?php

function hexToStr($hex){
    $string='';
    for ($i=0; $i < strlen($hex)-1; $i+=2){
        $string .= chr(hexdec($hex[$i].$hex[$i+1]));
    }
    return $string;
}

//функция генерации подписи по ключу и строке параметров
function checkSign($key, $req){
    $sign_hash = hash_hmac("sha1", $req, $key);
    $sign_tr = hexToStr($sign_hash);
    $sign = base64_encode($sign_tr);
    return $sign;
}

//Функция возвращает упорядоченную строку значений параметров POST-запроса
function getReqParams(){
    $reqparams = "";
    ksort($_POST);
    foreach ($_POST as $param => $valuep) {
        $reqparams = "$reqparams|$valuep";
    }
    return substr($reqparams,1);
}

//Извлечение цифровой подписи из заголовков запроса
function getSign(){
    $HEADERS = getallheaders();
    foreach ($HEADERS as $header => $value) {
       if ($header == 'X-Api-Signature') {
            $SIGN_REQ = $value;
       }
    }
    return $SIGN_REQ;
}

// Сортировка параметров
$Request = getReqParams();
// Пароль ishop для уведомлений магазина
$NOTIFY_PWD = "***";
// Вычисляем подпись
$reqres = checkSign($NOTIFY_PWD, $Request);

// Подпись из запроса
$SIGN_REQ = getSign();

if ($reqres == $SIGN_REQ) {
    $error = 0;
}
else $error = 151;

//Ответ
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

## Коды уведомлений  {#notify_codes}

Код завершения|Описание
--------------|--------
0|Успех
5|Ошибка формата параметров запроса
13|Ошибка соединения с базой данных
150|Ошибка проверки пароля
151|Ошибка проверки подписи
300|Ошибка связи с сервером

# Ответы

## Операции со счетами {#response_bill}

Возвращается объект с результатом выполнения операции.

<aside class="notice">
В ответ на два последовательных запроса с одинаковыми значениями параметров <i>{prv_id}</i>, <i>{bill_id}</i> и <i>amount</i> будет возвращаться один и тот же код результата выполнения операции.
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

Параметр|Тип|Описание
--------|---|--------
result_code|Integer|[Код результата](#errors)
bill_id|String|Уникальный идентификатор счета в системе провайдера
amount|String|Сумма счета, округленная до 2 или 3 знаков после запятой. Способ округления зависит от валюты.
originAmount|String|Сумма счета в исходной валюте счета (см. параметр `originCcy`), округленная до 2 или 3 знаков после запятой. Способ округления зависит от валюты.
ccy	|String|Идентификатор валюты (Alpha-3 ISO 4217 код)
originCcy|String|Идентификатор валюты выставленного счета (Alpha-3 ISO 4217 код)
status	|String|Текущий [статус счета](#status)
error	|Integer|Код ошибки
user|String|Идентификатор кошелька пользователя, которому выставлен счет (номер телефона в международном формате с префиксом "tel:")
comment|String|Комментарий к счету

## Операции с возвратами {#response_refund}

Возвращается объект с результатом выполнения операции возврата.
	 
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

Параметр|Тип|Описание
--------|---|--------
result_code|Integer|[Код результата](#errors)
refund_id|String|Уникальный идентификатор операции возврата счета в системе провайдера
amount|String|Сумма к возврату. Положительное число, округленное до 2 или 3 знаков после запятой. Способ округления зависит от валюты.
status	|String|Текущий [статус операции возврата](#status_refund)
error	|Integer|Код ошибки при проведении возврата платежа. В случае если сумма, переданная в запросе, превышает сумму самого счета либо сумму счета, оставшуюся после предыдущих возвратов, в ответе будет возвращен код ошибки 242.
user|String|Идентификатор кошелька пользователя, которому выставлен счет. Представляет собой номер телефона пользователя в международном формате с префиксом "tel:"

## Ответ в случае ошибки операции {#response_error}

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

Параметр|Тип|Описание
--------|---|--------
result_code|Integer|[Код результата](#errors)
description|String|Описание ошибки

# Статусы операций

## Статусы оплаты счетов {#status}

Статус|Описание|Финальный
------|--------|---------
waiting | Счет выставлен, ожидает оплаты| -
paid|Счет оплачен|+
rejected|Счет отклонен|+
unpaid|Ошибка при проведении оплаты. Счет не оплачен|+
expired	|Время жизни счета истекло. Счет не оплачен|+

## Статусы операции возврата {#status_refund}

Статус|Описание|Финальный
------|--------|---------
processing | Платеж в проведении| -
success|Платеж проведен|+
fail|Платеж неуспешен|+

# Список ошибок {#errors}

Код| Описание |Fatal*
---|----------|------
0|Успех	| 
5|Неверные данные в параметрах запроса|+
13|Сервер занят, повторите запрос позже|-
78|Недопустимая операция|+
150|Ошибка авторизации|+
152|Не подключен или отключен протокол|-
155|Данный идентификатор провайдера (API ID) заблокирован|+
210|Счет не найден|+
215|Счет с таким bill_id уже существует|+
241|Сумма слишком мала|+
242|Сумма слишком велика, или сумма, переданная в запросе возврата средств, превышает сумму самого счета либо сумму счета, оставшуюся после предыдущих возвратов|+
298|Кошелек с таким номером не зарегистрирован|+
300|Техническая ошибка|-
303|Неверный номер телефона|+
316|Попытка авторизации заблокированным провайдером|-
319|Нет прав на данную операцию|-
339|Ваш IP-адрес или массив адресов заблокирован|+
341|Обязательный параметр указан неверно или отсутствует в запросе|+
700|Превышен месячный лимит на операции|+
774|Кошелек временно заблокирован|-
1001|Запрещенная валюта для провайдера|+
1003|Не удалось получить курс конвертации для данной пары валют|-
1019|Не удалось определить сотового оператора для мобильной коммерции|+
1419|Нельзя изменить данные счета – он уже оплачивается или оплачен|+

* Признак означает, что при повторном запросе результат не изменится (ошибка не временная, проанализируйте данные запроса или обратитесь в Support).


