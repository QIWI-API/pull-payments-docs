# Уведомления об оплате счетов {#notification_ru}

###### Последнее обновление: 2018-05-03 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_notification_ru.html.md)

Уведомление представляет собой POST-запрос. Тело запроса содержит сериализованные данные счета в теле запроса (кодировка UTF-8), и параметр `command=bill`.

<h3 class="request method">Запрос → POST</h3>

> Пример

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
Чтобы начать получать уведомления, на сайте <a href="https://kassa.qiwi.com">QIWI Касса</a> достаточно указать адрес сервера для обработки уведомлений (раздел "Услуги", выбрать мерчанта, пункт "Настройки протокола"->"Серверные уведомления").

<ul class="nestedList notice_image">
    <li><h3>Подробнее</h3>
        <ul>
             <li><img src="/images/pull_rest_auth_kassa.png"/></li>
        </ul>
        <ul>
             <li><img src="/images/pull_rest_notifications_kassa.png"/></li>
        </ul>
        <ul>
            <li>Нажмите "Создать пароль и сохранить" для первичного получения <a href="#basic_notify">пароля</a> авторизации уведомлений</li>
        </ul>
    </li>
</ul>
</aside>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic *** - для авторизации по <a href="#basic_notify">логину/паролю</a></li>
             <li>X-Api-Signature: *** - для авторизации по <a href="#sign_notify">цифровой подписи</a></li>
             <li>Accept: text/xml</li>
             <li>Content-type: application/x-www-form-urlencoded</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>В теле запроса уведомления указываются параметры счета.</span>
    </li>
</ul>

Параметр|Описание|Тип|Обяз.
---------|--------|---|------
bill_id | Уникальный идентификатор счета в системе провайдера| String|+
status | Текущий [статус счета](#status)|String|+
amount | Сумма, на которую выставлялся счет, округленная до двух десятичных знаков | Number(6.2)|+
user | Номер QIWI Кошелька, на который был выставлен счет, с префиксом `tel:` | String|+
prv_name |  Наименование проекта, указанное на сайте [kassa.qiwi.com](https://kassa.qiwi.com) в разделе:"Настройки" | String|+
ccy | Идентификатор валюты (Alpha-3 ISO 4217 код) | String(3)|+
comment | Комментарий к счету | String(255)|+
command | `bill` - всегда по умолчанию | String |+

<aside class="notice">Так как для счета могут быть добавлены новые параметры на стороне QIWI Кошелька, то список параметров при обработке POST-запроса на стороне провайдера не должен фиксироваться.</aside>

<h3 class="request">Ответ ←</h3>

> Пример ответа на уведомление

~~~http
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
Если ответ с кодом результата 0 и HTTP-кодом 200 так и не был получен в течение суток после первой отправки, уведомления от системы QIWI Wallet прекращаются. На адрес электронной почты провайдера поступает письмо с новым статусом счета и указанием на возможную техническую неисправность в работе сервиса провайдера.
</aside>

<aside class="notice">
Для получения уведомлений провайдер должен принимать HTTP-запросы из следующих подсетей исключительно по портам 80, 443:

<li> 91.232.230.0/23</li>
<li> 79.142.16.0/20</li>
</aside>


## Авторизация уведомлений {#notifications_auth}

Для авторизации можно использовать [basic-авторизацию](#basic_notify) или [авторизацию подписи](#sign_notify). При запросах уведомлений на сервер провайдера также можно использовать SSL (в том числе самоподписанный сертификат). В HTTPS-запросах необходимо проверять серверный сертификат QIWI Wallet.

<aside class="notice">
Если сертификат для SSL-шифрования сгенерирован самостоятельно и не является доверенным со стороны стандартных центров сертификации, его необходимо загрузить в систему QIWI Wallet.

<ul class="nestedList notice_image">
   <li><h3>Сайт <a href="https://kassa.qiwi.com">QIWI Касса</a>, выбрать "Прикрепить цифровой сертификат" и сохранить настройки</h3>
        <ul>
           <li><img src="/images/pull_rest_notifications_kassa.png" /></li>
        </ul>
        <ul>
           <li><img src="/images/pull_rest_notifications_cert_kassa.png" /></li>
        </ul>
   </li>
</ul>

После загрузки данный сертификат будет считаться доверенным. Сертификат должен быть в одном из следующих форматов:
<ul><li>PEM (текстовый файл с расширением .pem: Privacy-enhanced Electronic Mail) - закодированный BASE64 сертификат DER, помещенный между строками <i>-----BEGIN CERTIFICATE-----</i> и <i>-----END CERTIFICATE-----</i>.</li>
<li>DER (файл с расширением.cer, .crt, .der) – обычно в бинарном формате DER, однако PEM сертификаты также допускаются с таким расширением.</li></ul>
</aside>

### Basic-авторизация {#basic_notify}

> Пример уведомления с Basic-авторизацией

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
Authorization: Basic ***
Host: service.ru

command=bill&bill_id=BILL-1&status=paid&error=0&amount=1.00&user=tel%3A%2B79031811737&prv_name=Retail_Store&ccy=RUB&comment=test
~~~

Логин равен [ID проекта](#auth_param).

Для первичного получения или смены пароля на сайте [QIWI Касса](https://kassa.qiwi.com), нажмите кнопку "Создать пароль и сохранить" или "Сменить пароль оповещения", соответственно.

<ul class="nestedList">
    <li><h3>Подробнее</h3>
        <ul>
             <li><img src="/images/pull_rest_notifications_pass_kassa.png" /></li>
        </ul>
        <ul>
             <li><img src="/images/pull_rest_notifications_cert_kassa.png" /></li>
        </ul>
    </li>
</ul>


### Авторизация по подписи {#sign_notify}

> Пример уведомления с подписью

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: text/xml
Content-type: application/x-www-form-urlencoded
X-Api-Signature: J4WNfNZd***V5mv2w=
Host: service.ru

command=bill&bill_id=LocalTest17&status=paid&error=0&amount=0.01&user=tel%3A%2B78000005122&prv_name=Test&ccy=RUB&comment=Some+Descriptor
~~~

Для использования этого способа авторизации уведомлений, на сайте <a href="https://kassa.qiwi.com">QIWI Касса</a> достаточно активировать флаг "Использовать HMAC подпись вместо basic-авторизации".

<ul class="nestedList notice_image">
   <li><h3>Подробнее</h3>
        <ul>
           <li><img src="/images/pull_rest_notifications_kassa_sign.png" /></li>
        </ul>
   </li>
</ul>

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

## Пример реализации {#notify_php}

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
// Пароль  для уведомлений магазина
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

Пример на языке PHP реализует авторизацию уведомлений системы QIWI Wallet с проверкой цифровой подписи. Откройте вкладку _PHP_ справа.

## Коды уведомлений  {#notify_codes}

Код завершения|Описание
--------------|--------
0|Успех
5|Ошибка формата параметров запроса
13|Ошибка соединения с базой данных
150|Ошибка проверки пароля
151|Ошибка проверки подписи
300|Ошибка связи с сервером
