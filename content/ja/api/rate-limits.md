---
title: レートリミット
description: Defining how often you can hit the REST API
menu:
  docs:
    weight: 20
    parent: api
---


レートリミット情報はレスポンスヘッダで返されます：

|Header|Description|
| :--- | :--- |
|`X-RateLimit-Limit`|一定期間に許可されたリクエストの数|
|`X-RateLimit-Remaining`|リクエストできる残数|
|`X-RateLimit-Reset`|レートリミットがリセットされるタイムスタンプ|

{{< hint style="info" >}}
APIメソッドは複数のレートリミットの影響を受ける可能性があります。ヘッダーは最も近い制限数を返します。
{{</ hint >}}

レートリミットの種類です：

|Endpoint|Bucket|Time period|Limit|Limit type|
| :--- | :--- | :--- | :--- | :--- |
|`* /api/*`|Account access|5 分|300|Account|
|`* /api/*`|IP access|5 分|300|IP|
|`POST /api/v1/accounts`|App sign-up|30 分|5|IP|
|`POST /api/v1/media`|Uploading|30 分|30|Account|
|`DELETE /api/v1/statuses/:id`|Deletion|30 分|30|Account|
|`POST /api/v1/statuses/:id/unreblog`|Deletion|30 分|30|Account|
