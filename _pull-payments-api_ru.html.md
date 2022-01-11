# Pull REST API {#pull-payments-api_ru}

###### Последнее обновление: 2018-05-03 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_pull-payments-api_ru.html.md)

## Последовательность операций {#steps}

<!--
<div class="mermaid">
sequenceDiagram
    participant Customer as Пользователь
    participant Merchant as Провайдер
    participant vqw as Сервис<br>QIWI Кошелек
    participant checkout as Платежная<br>страница QIWI
    Customer->>Merchant: Заказ товара
    activate Merchant
    Merchant->>vqw: Выставление счета
    activate vqw
    vqw->>Merchant: Информация о счете<br>и его статус
    opt Рекомендуется выполнить для улучшения конверсий оплаты
    Merchant-\->>Customer: Переадресация<br>на Платежную страницу
    deactivate Merchant
    Customer->>checkout: Оплата счета
    checkout->>vqw: Платежная<br>транзакция
    end
    Note over Customer, vqw: Оплата счета
    Customer->>vqw: Оплата счета на qiwi.com, QIWI Кошелек Android/iPhone<br>или в терминале QIWI
    vqw->>vqw: Проведение платежа
    opt Уведомление
    vqw->>Merchant: Уведомление о статусе счета
    activate Merchant
    Merchant-\->>vqw: OK
    deactivate vqw
    deactivate Merchant
    end
    Merchant->>Customer: Выполнение заказа
</div>
-->

![Operation Flow](/images/pullrest_1.png)

* Пользователь формирует заказ на сайте провайдера.

* Сервис провайдера [выставляет счет](#invoice_rest).

* Для повышения конверсии оплаты счетов рекомендуется перенаправлять пользователя на [платежную форму](#checkout_ru) QIWI Wallet. Также пользователь может оплатить счет через любой другой интерфейс QIWI Кошелька (сайт qiwi.com, платежный терминал QIWI, мобильное приложение Android или iOS).

* Если сервис провайдера использует [уведомления](#notification_ru), то после проведения платежа система QIWI Wallet высылает уведомление на сервер провайдера об оплате данного счета, либо, если пользователь отклонил счет, о неоплате. Уведомления об оплате счета содержат параметры авторизации, которые необходимо проверять на сервере провайдера.

* В любом случае, провайдер может:
  * [запросить текущий статус оплаты счета](#invoice-status),
  * [отменить счет](#cancel) (если пользователь еще не инициировал оплату).

* После подтверждения оплаты счета провайдер исполняет заказ пользователя.

## Средства для разработки {#node_sdk}

* [NODE JS SDK](https://github.com/QIWI-API/pull-payments-node-js-sdk) - Пакет готовых решений для разработки server2server интеграций для приема платежей на вашем сайте c помощью Node.js.


## Авторизация {#auth_param}

Все запросы мерчанта к Pull REST API авторизуются посредством HTTP basic-авторизации. Для авторизации используются API ID и API password. Заголовок представляет собой параметр `Authorization`, значение которого представлено как: `Basic Base64(API_ID:API_PASSWORD)`

~~~javascript
const prv_id = 21379721;
const api_id = 62573819;
const api_password = '**********';

const qiwiRestApi = new QiwiPullAPI(prv_id, api_id, api_password);
~~~

~~~shell
user@server:~$ curl "адрес сервера"
~~~

  <ul class="nestedList params">
    <li><h3>Авторизация и работа с формами</h3><span>Данные могут быть получены на сайте <a href='http://kassa.qiwi.com'>QIWI Кассы</a></span> 
     </li>
</ul>




Параметр|Описание|Тип|Обяз.
 ---------|--------|---|------
 API_ID | Идентификатор для авторизации провайдера в API | Integer| +
 API_PASSWORD | Пароль для авторизации в API| String | +
 ID проекта | Числовой идентификатор сервиса провайдера | Integer | +

<aside class="notice">
Получить служебные данные можно на сайте <a href='http://kassa.qiwi.com'>QIWI Кассы</a> в разделе "Настройки протокола - Аутентификационные данные", "Создать новый API ID".

<ul class="nestedList notice_image">
    <li><h3>Подробнее</h3>
        <ul>
             <li><img src="/images/pull_rest_auth_kassa.png" /></li>
        </ul>
    </li>
</ul>

</aside>

<aside class="notice">
Если у вас нет учетной записи в QIWI Кассе, то получить служебные данные можно на партнерском сайте <a href='http://kassa.qiwi.com'>kassa.qiwi.com</a> в разделе "Настройки", "Сгенерировать новый ID".

<ul class="nestedList notice_image">
    <li><h3>Подробнее</h3>
        <ul>
             <li><img src="/images/pull_rest_auth.png" /></li>
        </ul>
    </li>
</ul>

</aside>


## Выставление счета за покупку {#invoice_rest}

Запрос выставляет новый счет на указанный номер телефона (номер кошелька QIWI Wallet).

~~~php
<?php
//Пример реализации запроса на PHP
//Идентификатор магазина из вкладки "Данные магазина"
$SHOP_ID = "21379721";
//API ID из вкладки "Данные магазина"
$REST_ID = "62573819";
//API пароль из вкладки "Данные магазина"
$PWD = "**********";
//ID счета
$BILL_ID = "99111-ABCD-1-2-1";
$PHONE = "79191234567";

$data = array(
    "user" => "tel:+" . $PHONE,
    "amount" => "1000.00",
    "ccy" => "RUB",
    "comment" => "Товар из корзины",
    "lifetime" => "2015-01-30T15:35:00",
    "pay_source" => "qw",
    "prv_name" => "Магазин"
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
$url = 'https://oplata.qiwi.com/order/external/main.action?shop='.$SHOP_ID.'&
transaction='.$BILL_ID.'&successUrl=http%3A%2F%2Fieast.ru%2Findex.php%3Froute%3D
payment%2Fqiwi%2Fsuccess&failUrl=http%3A%2F%2Fieast.ru%2Findex.php%3Froute%3D
payment%2Fqiwi%2Ffail&pay_source=card';
echo '<br><br><b><a href="'.$url.'">Ссылка переадресации для оплаты счета</a></b>';
?>
~~~

~~~javascript
const bill_id = '99111-ABCD-1-2-1';

const fields = {
    amount: 1000.00,
    ccy: 'RUB',
    comment: 'Все очень хорошо',
    lifetime: '2015-01-30T15:35:00',
    user: 'tel:+79191234567',
    pay_source: 'qw',
    prv_name: 'Хороший магазин'
};

qiwiRestApi.createBill( bill_id, fields ).then( data => {
    //do with data
});
~~~

<h3 class="request method">Запрос → PUT</h3>

~~~shell
  user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/99111-ABCD-1-2-1"
    -X PUT --header "Accept: text/json"
    --header "Authorization: Basic ***"
    -d "user=tel%3A%2B79031234567&amount=10.00&ccy=RUB&comment=test&lifetime=2016-09-25T15:00:00"
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>В pathname PUT-запроса используются два параметра счета:</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор, который отображается в пункте "ID проекта" на партнерском сайте kassa.qiwi.com)</li>
             <li><strong>bill_id</strong> - уникальный идентификатор счета в системе провайдера. Уникальность означает, что идентификатор должен отличаться от идентификаторов всех ранее выставленных счетов провайдера.</li>
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
user | Идентификатор номера QIWI Wallet, на который выставляется счет (в международном формате), с префиксом `tel:` | String(20)|+
amount | Сумма, на которую выставляется счет. Округляется в меньшую сторону до двух десятичных знаков | Number(6.2)|+
ccy | Идентификатор валюты (Alpha-3 ISO 4217 код). Может использоваться любая валюта, предусмотренная договором с КИВИ | String(3)|+
comment | Комментарий к счету | String(255)|+
lifetime | Дата/время с точностью до секунд в формате ISO 8601 (`ГГГГ-ММ-ДДTчч:мм:сс`). Указывается московское время. Если счет не будет оплачен до этого времени, ему присваивается финальный статус и последующая оплата станет невозможна.<br> **Внимание! По истечении 45 суток от даты выставления счет автоматически будет переведен в финальный статус.**|dateTime|+
pay_source |`mobile` - оплата счета будет выполнена с баланса мобильного телефона пользователя, <br>`qw` – оплата любым способом через интерфейс QIWI Кошелька.<br> По умолчанию `qw` |String|-
prv_name|Название провайдера.| String(100)|-

Для просмотра примера PHP-скрипта выставления счета и перенаправления на Платежную форму QIWI откройте вкладку _PHP_ справа.

Данный пример иллюстрирует, когда и где в запросах к системе QIWI Wallet используются [авторизационные данные](#auth_param) провайдера, а именно, идентификатор магазина, API ID и пароль к API.


<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: text/json
{
  "response": {
     "result_code": 0,
     "bill": {
        "bill_id": "99111-ABCD-1-2-1",
        "amount": "1000.00",
        "ccy": "RUB",
        "status": "waiting",
        "error": 0,
        "user": "tel:+79191234567",
        "comment": "Счет от магазина"
     }
  }
}
~~~
~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~


Формат ответа зависит от заголовка "Accept" в исходном запросе:


<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
             <li>Accept: text/xml или Accept: application/xml - формат ответа XML</li>
        </ul>
    </li>
</ul>



<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>


Параметр|Тип|Описание
--------|---|--------
result_code|Integer|[Код результата](#errors)
description|String|Описание ошибки. Передается в случае ошибки (ненулевой `result_code`)
bill|Object|Данные о счете. Передается в случае успешного результата (`result_code` = 0). Параметры:
bill.bill_id|String|Уникальный идентификатор счета в системе провайдера
bill.amount | String| Сумма, на которую выставлен счет. Округляется в меньшую сторону до двух десятичных знаков.
bill.ccy|String|Идентификатор валюты (Alpha-3 ISO 4217 код)
bill.status|String|Текущий [статус счета](#status)
bill.error|Integer|Константа, всегда `0`
bill.user|String|Идентификатор кошелька пользователя, которому выставлен счет (номер телефона в международном формате с префиксом `tel:`)
bill.comment|String|Комментарий к счету


## Проверка статуса оплаты счета {#invoice-status}

Метод для проверки текущего статуса счета.

~~~javascript
const bill_id = '99111-ABCD-1-2-1';

qiwiRestApi.getStatus(bill_id).then( data => {
    //do with data
});
~~~

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/21379721/bills/99111-ABCD-1-2-1"
  --header "Authorization: Basic ***"
  --header "Accept: text/json"
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Параметры передаются в pathname.</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор, который отображается в пункте "ID проекта" на партнерском сайте kassa.qiwi.com)</li>
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

~~~http
HTTP/1.1 200 OK
Content-Type: text/json
{
  "response": {
     "result_code": 0,
     "bill": {
        "bill_id": "99111-ABCD-1-2-1",
        "amount": "1000.00",
        "ccy": "RUB",
        "status": "waiting",
        "error": 0,
        "user": "tel:+79031234567",
        "comment": "Счет от магазина"
     }
  }
}
~~~
~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
             <li>Accept: text/xml или Accept: application/xml - формат ответа XML</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>

Параметр|Тип|Описание
--------|---|--------
result_code|Integer|[Код результата](#errors)
description|String|Описание ошибки. Передается в случае ошибки (ненулевой `result_code`)
bill|Object|Данные о счете. Передается в случае успешного результата (`result_code` = 0). Параметры:
bill.bill_id|String|Уникальный идентификатор счета в системе провайдера
bill.amount |String| Сумма, на которую выставлен счет. Округляется в меньшую сторону до двух десятичных знаков.
bill.originAmount|String|Сумма счета в валюте баланса, с которого оплачивался счета (см. параметр `originCcy`). Округляется в меньшую сторону до двух десятичных знаков. **Возвращается только для счетов, по которым пользователь инициировал оплату**.
bill.ccy|String|Идентификатор валюты, в которой выставлен счет (Alpha-3 ISO 4217 код)
bill.originCcy|String|Идентификатор валюты баланса, с которого оплачивался счет (Alpha-3 ISO 4217 код). **Возвращается только для счетов, по которым пользователь инициировал оплату**.
bill.status|String|Текущий [статус счета](#status)
bill.error|Integer|Константа, всегда `0`
bill.user|String|Идентификатор кошелька пользователя, которому выставлен счет (номер телефона в международном формате с префиксом `tel:`)
bill.comment|String|Комментарий к счету



## Отмена неоплаченного счета {#cancel}

Метод для отмены неоплаченного клиентом счета до наступления предельной даты оплаты счета (см. [параметр](#invoice_rest) `lifetime`).

~~~javascript
const bill_id = '99111-ABCD-1-2-1';

qiwiRestApi.cancel(bill_id).then( data => {
    //do with data
});
~~~

<h3 class="request method">Запрос → PATCH</h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/99111-ABCD-1-2-1"
  -X PATCH
  --header "Authorization: Basic ***"
  --header "Accept: text/json"
  --header "Content-type: application/x-www-form-urlencoded; charset=utf-8"
  -d "status=rejected"
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Параметры передаются в pathname.</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор, который отображается в пункте "ID проекта" на партнерском сайте kassa.qiwi.com)</li>
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

~~~http
HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "bill": {
         "bill_id": "99111-ABCD-1-2-1",
         "amount": "1000.00",
         "ccy": "RUB",
         "status": "rejected",
         "error": 0,
         "user": "tel:+79031234567",
         "comment": "Счет от магазина"
      }
   }
}
~~~
~~~http
HTTP/1.1 401 Unauthorized
Content-Type: text/json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
             <li>Accept: text/xml или Accept: application/xml - формат ответа XML</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>

Параметр|Тип|Описание
--------|---|--------
result_code|Integer|[Код результата](#errors)
description|String|Описание ошибки. Передается в случае ошибки (ненулевой `result_code`)
bill|Object|Данные об отмененном счете. Передается в случае успешного результата (`result_code` = 0). Параметры:
bill.bill_id|String|Уникальный идентификатор счета в системе провайдера
bill.amount |String| Сумма, на которую выставлен счет. Округляется в меньшую сторону до двух десятичных знаков.
bill.ccy|String|Идентификатор валюты, в которой выставлен счет (Alpha-3 ISO 4217 код)
bill.status|String|Текущий [статус счета](#status)
bill.error|Integer|Константа, всегда `0`
bill.user|String|Идентификатор кошелька пользователя, которому выставлен счет (номер телефона в международном формате с префиксом `tel:`)
bill.comment|String|Комментарий к счету


## Возврат оплаченного счета {#refund}

Метод выполняет полный или частичный возврат средств по счету, оплаченному клиентом, на его QIWI Кошелек. При этом создается платеж, обратный платежу на оплату счета. Валюта платежа совпадает с валютой исходного счета.

По одному и тому же счету можно выполнять несколько операций возврата, при условии что:

* сумма всех операций возврата не превышает суммы исходного счета;
* для разных операций возврата одного счета используются разные идентификаторы.

<aside class="warning">
Если сумма, переданная в запросе, превышает сумму самого счета либо сумму счета, оставшуюся после предыдущих возвратов, в ответе будет возвращен код ошибки 242.
</aside>

### Последовательность операций

![Refund Operation Flow](/images/pullrest_2.png)

* Провайдер отправляет запрос на осуществление возврата.

* Если финальный статус возврата не получен в ответе, провайдер периодически опрашивает систему QIWI Wallet о [текущем статусе возврата](#refund_status) до получения финального статуса.

* Данный сценарий можно повторять несколько раз до тех пор, пока счет не будет полностью отменен (возвращена вся сумма).

~~~javascript
const bill_id = '893794793973';
const refund_id = '899343443';
const amount = 5.0;

qiwiRestApi.refund(bill_id, refund_id, amount).then( data => {
    //do with data
});
~~~

<h3 class="request method">Запрос → PUT</h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/893794793973/refund/899343443"
  -v -w "%{http_code}"
  -X PUT
  --header "Accept: text/json"
  --header "Authorization: Basic ***"
  --header "Content-type: application/x-www-form-urlencoded; charset=utf-8"
  -d "amount=5.0"
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a>/refund/<a>refund_id</a></span></h3>
        <ul>
        <strong>Параметры передаются в pathname.</strong>
              <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор, который отображается в пункте "ID проекта" на партнерском сайте kassa.qiwi.com)</li>
              <li><strong>bill_id</strong> - уникальный идентификатор счета в системе провайдера.</li>
              <li><strong>refund_id</strong> - идентификатор операции, уникальный в рамках операций возврата счета bill_id. Формат идентификатора: строка от 1 до 9 символов, содержащая только прописные или строчные латинские буквы (a-z, A-Z) и цифры от 0 до 9.</li>
         </ul>
          <ul>
          <strong>Параметр передается в body запроса.</strong>

             <li><strong>amount</strong> - сумма возврата. Должна быть меньше либо равна сумме счета <i>{bill_id}</i>, по которому производится возврат. Округляется в меньшую сторону до двух десятичных знаков.</li>
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

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": "899343443",
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
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
             <li>Accept: text/xml или Accept: application/xml - формат ответа XML</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>

Параметр|Тип|Описание
--------|---|--------
result_code|Integer|[Код результата](#errors)
description|String|Описание ошибки. Передается в случае ошибки (ненулевой `result_code`)
refund|Object|Данные об операции возврата. Передается в случае успешного результата (`result_code` = 0). Параметры:
refund.refund_id|String|Уникальный идентификатор операции возврата счета в системе провайдера
refund.amount|String|Сумма к возврату. Положительное число, округляется в меньшую сторону до двух десятичных знаков.
refund.status|String|Текущий [статус операции возврата](#status_refund)
refund.error|Integer|[Код ошибки](#errors) при проведении возврата платежа. В случае если сумма, переданная в запросе, превышает сумму самого счета либо сумму счета, оставшуюся после предыдущих возвратов, в ответе будет возвращен код ошибки `242`.


## Проверка статуса возврата {#refund_status}

Метод для проверки текущего статуса операции возврата средств по оплаченному счету.

~~~javascript
const bill_id = '893794793973';
const refund_id = '899343443';

qiwiApi.getRefundStatus(bill_id, refund_id).then( data => {
    //do with data
});
~~~

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/893794793973/refund/899343443"
  -v -w "%{http_code}"
  --header "Accept: text/json"
  --header "Authorization: Basic ***"
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a>/refund/<a>refund_id</a></span></h3>
        <ul>
        <strong>Параметры передаются в pathname.</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор, который отображается на партнерском сайте kassa.qiwi.com)</li>
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

~~~http
HTTP/1.1 200 OK
Content-Type: text/json
{
   "response": {
      "result_code": 0,
      "refund": {
         "refund_id": "899343443",
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
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
             <li>Accept: text/xml или Accept: application/xml - формат ответа XML</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>

Параметр|Тип|Описание
--------|---|--------
result_code|Integer|[Код результата](#errors)
description|String|Описание ошибки. Передается в случае ошибки (ненулевой `result_code`)
refund|Object|Данные об операции возврата. Передается в случае успешного результата (`result_code` = 0). Параметры:
refund.refund_id|String|Уникальный идентификатор операции возврата счета в системе провайдера
refund.amount|String|Сумма к возврату. Положительное число, округляется в меньшую сторону до двух десятичных знаков.
refund.status|String|Текущий [статус операции возврата](#status_refund)
refund.error|Integer|[Код ошибки](#errors) при проведении возврата платежа. В случае если сумма, переданная в запросе, превышает сумму самого счета либо сумму счета, оставшуюся после предыдущих возвратов, в ответе будет возвращен код ошибки 242.


## Статусы операций {#status}

###### Последнее обновление: 2017-07-11 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_pull-payments-api_ru.html.md)

### Статусы оплаты счетов {#status_bills}

Статус|Описание|Финальный
------|--------|---------
waiting | Счет выставлен, ожидает оплаты или оплачивается| -
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

## Список ошибок {#errors}

###### Последнее обновление: 2017-07-11 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_pull-payments-api_ru.html.md)

Код| Описание |Fatal*
---|----------|------
0|Успех	|
5|Неверные данные в параметрах запроса|+
13|Сервер занят, повторите запрос позже|-
78|Недопустимая операция|+
150|Ошибка авторизации|+
152|Не подключен или отключен протокол|-
155|Данный идентификатор провайдера ([API ID](#auth_param)) заблокирован|+
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
