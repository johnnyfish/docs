---
title: Troubleshoot Replication Zone Configurations
summary: Troubleshooting guide for replication zones, which control the number and location of replicas for specific sets of data.
keywords: ttl, time to live, availability zone
toc: true
docs_area: manage
---

## Overview

There are several classes of common problems users encounter when [manually configuring replication zones]({% link {{ page.version.version }}/configure-replication-zones.md %}#manage-replication-zones). Adding zone configurations manually can be tedious and it is easy to introduce logic errors.

Generally, the problems tend to fall into one of the following categories:

- "the system isn't sending replicas where I told it to"
- "the system isn't managing replicas how I told it to"

This behavior is almost always caused by a replication zone **misconfiguration**, but it can be difficult to see what the error is or how it was introduced. Zone configurations do not have much observability beyond `SHOW ZONE CONFIGURATIONS`, nor is there much built-in validation to prevent logic errors. It's easy to put the system in a state where you've told it to do two mutually incompatible things.

## Zone config inheritance

The hierarchy of inheritance for zone configs is as follows:

```
- default
  - database
    - table
      - index 
        - partition (row)
          - (sub)partition
```

Another way to look at this inheritance tree is as follows:


```
                                                         ┌───────────────────────────┐
                                                         │                           │
                           ┌────────────────────┐        │          Cluster          │─────────────────┐
                           │                    │        │                           │                 │
                           ▼                    │        └───────────────────────────┘                 │
             ┌──────────────────────────┐       │                      │                               │
             │                          │       │                      │                               │
             │         Database         │       └──────────────────────┘                               │
             │                          │                                                              ▼
             └──────────────────────────┘                                                ┌───────────────────────────┐
                           │                                                             │                           │
                           │                                                           ┌─│         Database          │──┐
                           ▼                                                           │ │                           │  │
             ┌───────────────────────────┐                                             ▼ └───────────────────────────┘  │
             │                           │                               ┌───────────────────────────┐                  │
             │           Table           │                               │                           │                  │
             │                           │                               │           Table           │       ┌──────────┘
             └───────────────────────────┘                               │                           │       │
                           │                                             └───────────────────────────┘       ▼
                           │                                                           │       ┌───────────────────────────┐
                           ▼                                                           │       │                           │
             ┌───────────────────────────┐                                             │       │           Table           │
             │                           │           ┌───────────────────────────┐     │       │                           │
             │            ...            │           │                           │     │       └───────────────────────────┘
             │                           │           │           Index           │◀────┤                     │
             └───────────────────────────┘           │                           │     │                     │
                                                     └───────────────────────────┘     │                     │
                                                                                       │                     │
                                                                                       │                     │
                                                     ┌───────────────────────────┐     │                     │
                                                     │                           │     │                     │          ┌───────────────────────────┐
                                                     │         Partition         │◀────┘                     │          │                           │
                                                     │                           │                           ├─────────▶│           Index           │
                                                     └───────────────────────────┘                           │          │                           │
                                                                   │                                         │          └───────────────────────────┘
                                                                   │                                         │
                                                                   │                                         │
                                                                   ▼                                         │
                                                     ┌───────────────────────────┐                           │          ┌───────────────────────────┐
                                                     │                           │                           │          │                           │
                                                     │      (Sub)Partition       │                           ├─────────▶│           Index           │
                                                     │                           │                           │          │                           │
                                                     └───────────────────────────┘                           │          └───────────────────────────┘
                                                                                                             │
                                                                                                             │          ┌───────────────────────────┐
                                                                                                             │          │                           │
                                                                                                             └─────────▶│         Partition         │
                                                                                                                        │                           │
                                                                                                                        └───────────────────────────┘
```

The most common class of logic error occurs because of the way inheritance works for replication zone configurations. As discussed in [Replication Controls]({% link {{ page.version.version }}/configure-replication-zones.md %}#level-priorities), CockroachDB always uses the most granular replication zone available in a "bottom-up" fashion.

When you manually set a field at, say, the table level, it overrides the value that was already set at the next level up, in the parent database. If you later change something at the database level and find that it isn't working as expected for all tables, it may be that the more-specific settings you applied at the table level are overriding the database-level settings. In other words, the system is doing what it was told, because it is respecting the table-level change you already applied. However, this may not be what you _intended_.

Each newly created zone config inherits all of its initial values from its parent object. Thus, any changes to a zone configuration must be supplied by the user.

Furthermore, a zone config at some level of the inheritance tree only stores the values that differ from its parent. Otherwise, it looks up its values for any unset fields in the parent object's zone configuration. This continues recursively up the tree all the way to the `default` zone config (Logically speaking - there is caching involved).

Note that each partition can be further split into subpartitions, etc. with the following difference in inheritance behavior: each subpartition, instead of inheriting its fields from the parent partition, inherits its fields from the parent table.

## Replication system priorities

A further wrinkle is that the [Replication Layer]({% link {{ page.version.version }}/architecture/replication-layer.md %})'s top priority is avoiding data loss, _not_ putting replicas exactly where you told it to. For more information about this limitation, see [the data domiciling docs]({% link {{ page.version.version }}/data-domiciling.md %}#known-limitations).

That said, the replication logic takes user-supplied zone configurations into account when allocating replicas. 

## Types of problems

As noted previously, the problems tend to fall into one of the following general categories:

- "the system isn't sending replicas where I told it to"
- "the system isn't doing what I told it to with the replica configuration"

Specifically:

- If the problem is "the system isn't sending replicas where I told it to", see:
    - `constraints = '{+region=europe-west1: 1, +region=us-east1: 1, +region=us-west1: 1}',`
    - `voter_constraints = '[+region=us-east1]',`
    - `lease_preferences = '[[+region=us-east1]]'`
- If the problem is "the system isn't doing what I told it to with the replica configuration", see:
    - `num_replicas = 5,`
    - `range_min_bytes = 134217728,`
    - `range_max_bytes = 536870912,`
    - `gc.ttlseconds = 14400,`
    - `num_voters = 3,`

The most common reason why "the thing isn't going where I told it to go" or "the thing isn't doing what I told it to do" is misconfiguration.

## Troubleshooting steps

Troubleshooting zone configs is difficult because it requires running a mix of one or more of the following statements, peering at their output, and deducing what went wrong. Specifically, running a mix of:

- [`SHOW ZONE CONFIGURATIONS`]({% link {{ page.version.version }}/show-zone-configurations.md %}) for different levels of objects in the inheritance hierarchy and checking where they differ.
- [`SHOW ALL ZONE CONFIGURATIONS`]({% link {{ page.version.version }}/show-zone-configurations.md %}#view-all-replication-zones) and parsing the output into a tree-like format that lets you see what has changed. ([XXX](): is this really what we want to say?)

[XXX](): WRITE ME

### Run SQL statements

### Just wait longer

Sometimes the changes you make to a zone configuration are not reflected in the running system for a few minutes. Depending on the size of the cluster, this is expected behavior. It can take several minutes for changes to replica locality you specify in a changed zone config to propagate across a cluster. In general, the larger the cluster, the longer this process may take, due to the amount of data shuffling that occurs. In general, it's better to avoid making big changes to replica constraints on a large cluster unless you are prepared for it to take some time.

For more information about how to troubleshoot replication issues, especially under-replicated ranges, see [Troubleshoot Self-Hosted Setup > Replication issues]({% link {{ page.version.version }}/cluster-setup-troubleshooting.md %}#replication-issues).

## See also

- [Troubleshooting overview]({% link {{ page.version.version }}/troubleshooting-overview.md %})
- [Troubleshoot Self-Hosted Setup > Replication issues]({% link {{ page.version.version }}/cluster-setup-troubleshooting.md %}#replication-issues)
- [Replication Controls]({% link {{ page.version.version }}/configure-replication-zones.md %})
- [Critical nodes status endpoint]({% link {{ page.version.version }}/monitoring-and-alerting.md %}#critical-nodes-endpoint): check the status of your cluster's data replication, data placement, and zone constraint conformance.
