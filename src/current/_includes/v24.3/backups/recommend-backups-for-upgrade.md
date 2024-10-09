Because CockroachDB is designed with high fault tolerance, backups are primarily needed for [disaster recovery]({% link {{ page.version.version }}/disaster-recovery-planning.md %}). However, taking regular backups of your data is an operational best practice. When upgrading to a major release, **we recommend [taking a backup]({% link {{ page.version.version }}/backup-and-restore-overview.md %}) of your cluster**.