# Уведомления об оплате счетов {#notification_ru}

###### Последнее обновление: 2017-10-20 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_notification_ru.html.md)

Уведомление представляет собой POST-запрос. Тело запроса содержит сериализованные данные счета в теле запроса (кодировка UTF-8), и параметр `command=bill`.

<h3 class="request method">Запрос → POST</h3>

~~~shell
Пример

user@server:~$ curl "https://service.ru/qiwi-notify.php"
  -v -w "%{http_code}"
  -X POST --header "Accept: text/xml"
  --header "Content-Type: application/x-www-form-urlencoded; charset=utf-8"
  --Authorization: "Basic MjA0Mjp0ZXN0Cg=="
  -d "bill_id=BILL-1%26status=paid%26pay_date=2016%3A11%3A16T11%3A00%3A15%26amount=1.00%26user=tel%3A%2B79031811737%26prv_name=TEST%26ccy=RUB%26comment=test%26command=bill"
~~~

<ul class="nestedList url">
    <li><h3>URL</h3>
    </li>
</ul>

<aside class="notice">
Для получения уведомлений активируйте их на партнерском сайте ishop.qiwi.com (раздел "Настройки Pull (REST) протокола", пункт "Включить уведомления").
<ul class="nestedList notice_image">
    <li><h3>Подробнее</h3>
        <ul>
             <li><img src="/images/pull_rest_notifications.png"/></li>
        </ul>
    </li>
</ul>

Aдрес сервера для уведомлений указывается в том же разделе.

<ul class="nestedList notice_image">
   <li><h3>Подробнее</h3>
        <ul>
           <li><img src="/images/pull_rest_notification_url.png" /></li>
        </ul>
   </li>
</ul>
</aside>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic *** - авторизация по <a href="#basic_notify">логину/паролю</a></li>
             <li>X-Api-Signature: *** - авторизация по <a href="#sign_notify">цифровой подписи</a></li>
             <li>Accept: text/xml</li>
             <li>Content-type: application/x-www-form-urlencoded</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>В теле запроса указываются параметры счета.</span>
    </li>
</ul>

Параметр|Описание|Тип|Обяз.
---------|--------|---|------
status | Текущий [статус счета](#status)|String|+
amount | Сумма, на которую выставлялся счет, округленная до двух десятичных знаков | Number(6.2)|+
user | Номер Visa QIWI Кошелька, на который был выставлен счет | String|+
pay_date | Дата оплаты счета (заполняется, если счет оплачен)|DateTime (`YYYY-MM-DDThh:mm:ss`)|+
prv_name |  Наименование проекта, указанное на сайте ishop.qiwi.com в разделе:"Настройки"->"Данные проекта"->"Короткое наименование" | String|+
ccy | Идентификатор валюты (Alpha-3 ISO 4217 код) | String(3)|+
comment | Комментарий к счету | String(255)|+
command | `bill` - всегда по умолчанию | String |+

<h3 class="request">Ответ ←</h3>

~~~xml
HTTP/1.1 200 OK
Content-Type: text/xml

<?xml version="1.0"?>
<result>
<result_code>0</result_code>
</result>
~~~

Ответ на запрос должен быть в формате XML.

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Content-type: text/xml</li>
        </ul>
    </li>
</ul>


<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>


* `result` - Группирующий тег. Содержит описание результата обработки уведомления.
* `result_code` - Код результата обработки уведомления (целое положительное число). Рекомендуется возвращать коды результата в соответствии с [таблицей кодов завершения](#notify_codes).


<aside class="warning">
Если в ответе код результата обработки уведомления отличается от 0 и/или HTTP-код отличается от 200 (OK), это интерпретируется как временная ошибка провайдера. Сервер повторяет запрос с нарастающим интервалом в течение суток (<b>не более 50 попыток</b>) до получения в ответе кода результата 0 и кода состояния HTTP 200.
</aside>

<aside class="notice">
Если ответ с кодом результата 0 и HTTP-кодом 200 так и не был получен в течение суток после первой отправки, уведомления от сервера Visa QIWI Wallet прекращаются. На адрес электронной почты провайдера поступает письмо с новым статусом счета и указанием на возможную техническую неисправность в работе сервиса провайдера.
</aside>

<aside class="notice">
Для получения уведомлений провайдер должен принимать HTTP-запросы из следующих подсетей исключительно по портам 80, 443:

<li> 91.232.230.0/23</li>
<li> 79.142.16.0/20</li>
</aside>


## Авторизация уведомлений {#notifications_auth}

Для авторизации можно использовать [basic-авторизацию](#basic_notify) или [авторизацию подписи](#sign_notify). При запросах уведомлений на сервер провайдера также можно использовать SSL (в том числе самоподписанный сертификат). В HTTPS-запросах необходимо проверять серверный сертификат Visa QIWI Wallet.

<aside class="notice">
Если сертификат для SSL-шифрования сгенерирован самостоятельно и не является доверенным со стороны стандартных центров сертификации, его необходимо загрузить в систему Visa QIWI Wallet через инструмент <b>Сертификат</b> раздела <b>Протоколы - REST-протокол</b> сайта ishop.qiwi.com.

<ul class="nestedList notice_image">
   <li><h3>Подробнее</h3>
        <ul>
           <li><img src="/images/pull_rest_notification_cert.png" /></li>
        </ul>
   </li>
</ul>

После загрузки данный сертификат будет считаться доверенным. Сертификат должен быть в одном из следующих форматов:
<ul><li>PEM (текстовый файл с расширением .pem) – (Privacy-enhanced Electronic Mail) закодированный BASE64 сертификат DER, помещенный между строками <i>-----BEGIN CERTIFICATE-----</i> и <i>-----END CERTIFICATE-----</i>.</li>
<li>DER (файл с расширением.cer, .crt, .der) – обычно в бинарном формате DER, однако PEM сертификаты также допускаются с таким расширением.</li></ul>
</aside>

## Basic-авторизация {#basic_notify}

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
Authorization: Basic ***
Host: service.ru

command=bill&bill_id=BILL-1&status=paid&error=0&amount=1.00&user=tel%3A%2B79031811737&prv_name=Retail_Store&ccy=RUB&comment=test
~~~

Логин равен [ID магазина](#auth_param). Для получения пароля нажмите кнопку "Сменить пароль оповещения" в [Личном кабинете мерчанта](http://ishop.qiwi.com)в разделе "Настройки"->"REST-Протокол".

<ul class="nestedList">
    <li><h3>Подробнее</h3>
        <ul>
             <li><img src="/images/pull_rest_notifications_pass.png" /></li>
        </ul>
    </li>
</ul>


## Авторизация по подписи {#sign_notify}

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
X-Api-Signature: J4WNfNZd***V5mv2w=
Host: service.ru

command=bill&bill_id=LocalTest17&status=paid&error=0&amount=0.01&user=tel%3A%2B78000005122&prv_name=Test&ccy=RUB&comment=Some+Descriptor
~~~

<aside class="notice">
Для использования этого способа активируйте флаг "Подпись" на партнерском сайте ishop.qiwi.com в разделе <b>Протоколы - REST-протокол</b>.

<ul class="nestedList notice_image">
   <li><h3>Подробнее</h3>
        <ul>
           <li><img src="/images/pull_rest_notification_cert.png" /></li>
        </ul>
   </li>
</ul>
</aside>

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
   где:

   * `Notification_password_bytes` – ключ функции (байт-представление basic-пароля для уведомлений);
   * `Invoice_parameters_bytes` – байт-представление тела POST-запроса;
   * `hash` – результат хэш-функции.

4. HMAC-хэш преобразовать из строк в байты с использованием кодировки UTF-8 и base64-преобразовать.
5. Сравнить значение заголовка `X-Api-Signature` с результатом 4.

## Пример реализации

Пример на языке PHP реализует авторизацию уведомлений Visa QIWI Wallet с проверкой цифровой подписи. Откройте вкладку _PHP_ справа.

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