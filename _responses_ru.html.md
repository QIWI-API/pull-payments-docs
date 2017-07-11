# Ответы Pull REST API

###### Последнее обновление: 2017-07-11 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_responses_ru.html.md)


## Операции со счетами {#responses_ru}

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

<a href="#" onclick="history.back(); return false">Назад</a>

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
    "refund_id": "122swbill",
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

<a href="#" onclick="history.back(); return false">Назад</a>

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

<a href="#" onclick="history.back(); return false">Назад</a>


