The number of RUs consumed by KV writes, broken down by bytes. SQL statements are translated into lower-level KV write requests that are sent in batches. Batches may contain any number of requests. Requests can have a payload containing any number of bytes. Write operations are replicated to multiple [storage processes]({% link cockroachcloud/resource-usage-basic.md %}#understand-resource-consumption) (3 by default), with each replica counted as a separate write operation. Storage layer I/O is converted to Request Units using this equivalency:
<br>
<b>1 RU = 1 KiB write request payload (prorated)</b>
<br>
Correlate this metric with [Request Units (RUs)](#tenant.consumption.request_units). To learn more about how RUs are calculated, refer to [Resource Usage]({% link cockroachcloud/resource-usage-basic.md %}).