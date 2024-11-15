CockroachDB offers an improved leasing system rebuilt atop a stronger form of [Raft]({% link {{ page.version.version }}/architecture/replication-layer.md %}#raft) leadership that ensures that the Raft leader is **always** the range's leaseholder. This new type of lease is called a _Leader lease_, and supersedes [epoch-based leases]({% link {{ page.version.version }}/architecture/replication-layer.md %}#epoch-based-leases-table-data) and [expiration-based leases]({% link {{ page.version.version }}/architecture/replication-layer.md %}#expiration-based-leases-meta-and-system-ranges) leases while combining the performance of the former with the resilience of the latter. **Leader leases are not enabled by default.**