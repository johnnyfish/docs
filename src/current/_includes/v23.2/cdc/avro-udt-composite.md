A changefeed in [Avro format]({% link {{ page.version.version }}/changefeed-messages.md %}#avro) will not be able to serialize [user-defined composite (tuple) types](create-type.html). [Tracking GitHub Issue](https://github.com/cockroachdb/cockroach/issues/102903)