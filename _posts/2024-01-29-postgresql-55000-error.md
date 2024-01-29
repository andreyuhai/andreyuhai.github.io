---
title: PostgreSQL 55000 error
date: 2024-01-29
tags: [til, postgresql]
description: What was the root cause of PostgreSQL 55000 error in our case
---

We had a Kafka consumer which was crashing with the error below

```elixir
(Postgrex.Error ERROR 55000 (object_not_in_prerequisite_state) recovery is in progress

    hint: WAL control functions cannot be executed during recovery.)
```

In that consumer module, there was a function which was eventually trying to get the LSN from the DB by querying something like

```
SELECT pg_last_wal_replay_lsn()::varchar
```

It turns out we were actually querying a replica where WAL was enabled, figured that out quite quickly by asking in #postgresql IRC.
(_Well, if you're querying the LSN it obviously means you've got WAL enabled, it's even in the function name! But didn't know it back then. ¯\_(ツ)_/¯_)

> 10:49 < Berge> Note that a replica (using WAL replication) is considered to be in recovery

Thank you Berge! TIL!
