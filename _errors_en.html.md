# Error Codes {#errors}

<aside class="notice"><b>Fatal</b> means the result will not change with the second and subsequent requests (error is not temporary)</aside> 

Code| Description |Fatal
---|----------|------
0|Success	| Unrelated
5|Incorrect data in the request parameters|Y
13|Server is busy, try again later|N
78|Operation is forbidden|Y
150|Authorization error (e.g. invalid login/password)|Y
152|Protocol is not enabled or protocol is disabled|N
155|This merchant’s identifier ([API ID](#auth)) is blocked|Y
210|Invoice not found|Y
215|Invoice with this `bill_id` already exists|Y
241|Invoice amount is less than allowed|Y
242|Invoice amount is greater than allowed. Also returns to refund request when the amount of refund exceeds the initial invoice amount or the amount left after the previous refunds | Y
298|User not registered|Y
300|Technical error|N
303|Wrong phone number|Y
316|Authorization from the blocked merchant|N
319|No rights for the operation|N
339|IP-addresses blocked|Y
341|Required parameter is incorrectly specified or absent in the request|Y
700|Monthly limit on operations is exceeded|Y
774|Visa QIWI Wallet user account temporarily blocked|Y
1001|Currency is not allowed for the merchant|Y
1003|No convert rate for these currencies|N
1019|Unable to determine wireless operator for MNO balance payment|Y
1419|Bill was already payed|Y


