- When running the [`ALTER TYPE`]({% link {{ page.version.version }}/alter-type.md %}) statement, you can only reference a user-defined type from the database that contains the type.
- You can only [cancel]({% link {{ page.version.version }}/cancel-job.md %}) `ALTER TYPE` [schema change jobs]({% link {{ page.version.version }}/online-schema-changes.md %}) that drop values. This is because when you drop a value, CockroachDB searches through every row that could contain the type's value, which could take a long time. All other `ALTER TYPE` schema change jobs are [non-cancellable]({% link {{ page.version.version }}/cancel-job.md %}#known-limitations).