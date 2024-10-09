To restrict a user's access to changefeed data and sink credentials, enable the `changefeed.permissions.require_external_connection_sink.enabled` cluster setting. When you enable this setting, users with the [`CHANGEFEED` privilege]({% link {{ page.version.version }}/create-changefeed.md %}#required-privileges) on a set of tables can only create changefeeds into [external connections]({% link {{ page.version.version }}/create-external-connection.md %}).