# Operation Statuses

###### Last update: 2017-07-11 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_statuses_en.html.md)

## Invoice Status {#status}

Status|Description|Final
------|--------|---------
waiting | Invoice issued, pending payment| N
paid|Invoice has been paid|Y
rejected|Invoice has been rejected|Y
unpaid|Payment processing error. Invoice has not been paid|Y
expired	|Invoice expired. Invoice has not been paid|Y

## Refund Status {#status_refund}

Status|Description|Final
------|--------|---------
processing | Payment refund is pending| N
success|Payment refund is successful|Y
fail|Payment refund is unsuccessful|Y

