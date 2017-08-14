---
title: QIWI Wallet Pull REST API 2.1

search: true

metatitle: QIWI Wallet Pull REST API 2.1

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

includes:
  - pull-payments/pull-payments-api_ru
  - pull-payments/webform_ru
  - pull-payments/checkout_ru
  - pull-payments/notification_ru
  - pull-payments/responses_ru
  - pull-payments/statuses_ru
  - pull-payments/errors_ru

---

# Введение {#introduction}

###### Последнее обновление: 2017-05-31 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/pull-payments_ru.html.md)

QIWI Wallet Pull API открывает доступ к операциям со счетами в Visa QIWI Wallet из вашего приложения. Поддерживаются следующие операции:

* выставление счетов
* отмена неоплаченных счетов
* возврат средств по оплаченным счетам (пользователь отказался от услуги)
* проверка статуса выполнения операции
* оплата счета на Платежной форме QIWI

## Способы оплаты {#payment_methods}

* Пользователи могут оплачивать счета Visa QIWI Wallet
  * в интерфейсах платежной формы QIWI (bill.qiwi.com)
  * на сайте [qiwi.com](#https://qiwi.com)
    * с баланса своего Visa QIWI Wallet
    * с баланса мобильного телефона или с любой карты Visa/MasterCard.
  * в мобильных приложений QIWI
    * с баланса своего Visa QIWI Wallet
    * с баланса мобильного телефона или с любой карты Visa/MasterCard.
  * также доступна оплата счетов наличными в QIWI Терминалах.

**Данное API можно использовать только после [регистрации и подключения](https://ishop.qiwi.com).**

## Способы интеграции {}

Для работы с QIWI Wallet Pull API доступны следующие способы:

* [Веб-форма выставления счета](#webform_ru) - Не требует сложной реализации и дополнительной переадресации пользователя на Платежную форму. Ограничена по функциональности - поддерживает только выставление счета. Вызов веб-формы авторизуется по [API_ID](#auth_param) и цифровой подписи запроса, а также может выполняться без авторизации (не рекомендуется).

* [Pull REST API](#pull-payments-api_ru) - Полнофункциональное API для всех операций со счетами.

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
Получить служебные данные можно на партнерском сайте <a href='http://ishop.qiwi.com'>ishop.qiwi.com</a> в разделе "Протоколы - REST-протокол - Аутентификационные данные".

<ul class="nestedList notice_image">
    <li><h3>Подробнее</h3>
        <ul>
             <li><img src="images/pull_rest_auth.png" /></li>
        </ul>
    </li>
</ul>

</aside>

<a href="#" onclick="history.back(); return false">Назад</a>
