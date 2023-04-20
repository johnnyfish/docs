 Option                                                          | Value                   | Description
-----------------------------------------------------------------+-------------------------+------------------------------
`revision_history`<a name="with-revision-history"></a>           | [`BOOL`](bool.html) / None                     | Create a backup with full [revision history](take-backups-with-revision-history-and-restore-from-a-point-in-time.html), which records every change made to the cluster within the garbage collection period leading up to and including the given timestamp.<br><br>You can specify a backup with revision history without any value e.g., `WITH revision_history`. Or, you can explicitly define `WITH revision_history = 'true' / 'false'`. `revision_history` defaults to `true` when used with `BACKUP` or `CREATE SCHEDULE FOR BACKUP`. A value is **required** when using [`ALTER BACKUP SCHEDULE`](alter-backup-schedule.html).
`encryption_passphrase`<a name="with-encryption-passphrase"></a> | [`STRING`](string.html) | The passphrase used to [encrypt the files](take-and-restore-encrypted-backups.html) (`BACKUP` manifest and data files) that the `BACKUP` statement generates. This same passphrase is needed to decrypt the file when it is used to [restore](take-and-restore-encrypted-backups.html) and to list the contents of the backup when using [`SHOW BACKUP`](show-backup.html). There is no practical limit on the length of the passphrase.
`detached`<a name="detached"></a>                           | [`BOOL`](bool.html) / None                   |  When a backup runs in `detached` mode, it will execute asynchronously. The job ID will be returned after the backup [job creation](backup-architecture.html#job-creation-phase) completes. Note that with `detached` specified, further job information and the job completion status will not be returned. For more on the differences between the returned job data, see the [example](backup.html#run-a-backup-asynchronously). To check on the job status, use the [`SHOW JOBS`](show-jobs.html) statement. Backups running on a [schedule](create-schedule-for-backup.html) have the `detached` option applied implicitly.<br><br>To run a backup within a [transaction](transactions.html), use the `detached` option.
`kms`                                                            | [`STRING`](string.html) |  The [key management service (KMS) URI](take-and-restore-encrypted-backups.html#uri-formats) (or a [comma-separated list of URIs](take-and-restore-encrypted-backups.html#take-a-backup-with-multi-region-encryption)) used to encrypt the files (`BACKUP` manifest and data files) that the `BACKUP` statement generates. This same KMS URI is needed to decrypt the file when it is used to [restore](take-and-restore-encrypted-backups.html#restore-from-an-encrypted-amazon-s3-backup) and to list the contents of the backup when using [`SHOW BACKUP`](show-backup.html). <br/><br/>AWS, Azure, and Google Cloud KMS are supported.
`incremental_location`<a name="incr-location"></a> | [`STRING`](string.html) | Create an incremental backup in a different location than the default incremental backup location. <br><br>`WITH incremental_location = 'explicit_incrementals_URI'`<br><br>See [Incremental backups with explicitly specified destinations](take-full-and-incremental-backups.html#incremental-backups-with-explicitly-specified-destinations) for usage.