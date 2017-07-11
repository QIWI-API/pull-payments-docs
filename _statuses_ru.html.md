# Статусы операций

###### Последнее обновление: 2017-07-11 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_statuses_ru.html.md)

## Статусы оплаты счетов {#status}

Статус|Описание|Финальный
------|--------|---------
waiting | Счет выставлен, ожидает оплаты| -
paid|Счет оплачен|+
rejected|Счет отклонен|+
unpaid|Ошибка при проведении оплаты. Счет не оплачен|+
expired	|Время жизни счета истекло. Счет не оплачен|+

<a href="#" onclick="history.back(); return false">Назад</a>

## Статусы операции возврата {#status_refund}

Статус|Описание|Финальный
------|--------|---------
processing | Платеж в проведении| -
success|Платеж проведен|+
fail|Платеж неуспешен|+

<a href="#" onclick="history.back(); return false">Назад</a>
