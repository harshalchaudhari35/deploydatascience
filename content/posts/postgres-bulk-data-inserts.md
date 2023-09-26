---
title: "Postgres Bulk Data Inserts"
date: 2023-09-26T09:41:51+01:00
draft: False
tags: [databases, Postgres]
description: A Definitive Complied Guide for Bulk Inserts to Postgres, complied overtime from my own experiences and taken from various sources 
---

### A Definitive Complied Guide for Bulk Inserts to Postgres

#### Taken from a stackoverflow answer as below[^1]

(Note that this answer is about bulk-loading data into an existing DB or to create a new one. If you're interested DB restore performance with pg_restore or psql execution of pg_dump output, much of this doesn't apply since pg_dump and pg_restore already do things like creating triggers and indexes after it finishes a schema+data restore).

There's lots to be done. The ideal solution would be to import into an UNLOGGED table without indexes, then change it to logged and add the indexes. Unfortunately in PostgreSQL 9.4 there's no support for changing tables from UNLOGGED to logged. 9.5 adds ALTER TABLE ... SET LOGGED to permit you to do this.

If you can take your database offline for the bulk import, use pg_bulkload.

Otherwise:

- Disable any triggers on the table

- Drop indexes before starting the import, re-create them afterwards. (It takes much less time to build an index in one pass than it does to add the same data to it progressively, and the resulting index is much more compact).

- If doing the import within a single transaction, it's safe to drop foreign key constraints, do the import, and re-create the constraints before committing. Do not do this if the import is split across multiple transactions as you might introduce invalid data.

- If possible, use COPY instead of INSERTs

- If you can't use COPY consider using multi-valued INSERTs if practical. You seem to be doing this already. Don't try to list too many values in a single VALUES though; those values have to fit in memory a couple of times over, so keep it to a few hundred per statement.

- Batch your inserts into explicit transactions, doing hundreds of thousands or millions of inserts per transaction. There's no practical limit AFAIK, but batching will let you recover from an error by marking the start of each batch in your input data. Again, you seem to be doing this already.

- Use synchronous_commit=off and a huge commit_delay to reduce fsync() costs. This won't help much if you've batched your work into big transactions, though.

- INSERT or COPY in parallel from several connections. How many depends on your hardware's disk subsystem; as a rule of thumb, you want one connection per physical hard drive if using direct attached storage.

- Set a high max_wal_size value (checkpoint_segments in older versions) and enable log_checkpoints. Look at the PostgreSQL logs and make sure it's not complaining about checkpoints occurring too frequently.

- If and only if you don't mind losing your entire PostgreSQL cluster (your database and any others on the same cluster) to catastrophic corruption if the system crashes during the import, you can stop Pg, set fsync=off, start Pg, do your import, then (vitally) stop Pg and set fsync=on again. See WAL configuration. Do not do this if there is already any data you care about in any database on your PostgreSQL install. If you set fsync=off you can also set full_page_writes=off; again, just remember to turn it back on after your import to prevent database corruption and data loss. See non-durable settings in the Pg manual.

You should also look at tuning your system:

- Use good quality SSDs for storage as much as possible. Good SSDs with reliable, power-protected write-back caches make commit rates incredibly faster. They're less beneficial when you follow the advice above - which reduces disk flushes / number of fsync()s - but can still be a big help. Do not use cheap SSDs without proper power-failure protection unless you don't care about keeping your data.

- If you're using RAID 5 or RAID 6 for direct attached storage, stop now. Back your data up, restructure your RAID array to RAID 10, and try again. RAID 5/6 are hopeless for bulk write performance - though a good RAID controller with a big cache can help.

- If you have the option of using a hardware RAID controller with a big battery-backed write-back cache this can really improve write performance for workloads with lots of commits. It doesn't help as much if you're using async commit with a commit_delay or if you're doing fewer big transactions during bulk loading.

- If possible, store WAL (pg_wal, or pg_xlog in old versions) on a separate disk / disk array. There's little point in using a separate filesystem on the same disk. People often choose to use a RAID1 pair for WAL. Again, this has more effect on systems with high commit rates, and it has little effect if you're using an unlogged table as the data load target.

You may also be interested in Optimise PostgreSQL for fast testing.

--------

#### When using parquet source files

- Use `COPY table TO ... WITH BINARY` which is according to the documentation "is somewhat faster than the text and CSV formats." Only do this if you have millions of rows to insert, and if you are comfortable with binary data. 
- You can convert Parquet files to Binary using [pgpq](https://github.com/adriangb/pgpq)





### References
 [^1]: https://stackoverflow.com/questions/12206600/how-to-speed-up-insertion-performance-in-postgresql
 - https://www.postgresql.org/docs/current/populate.html
 - https://www.depesz.com/2007/07/05/how-to-insert-data-to-database-as-fast-as-possible/
 - https://github.com/adriangb/pgpq