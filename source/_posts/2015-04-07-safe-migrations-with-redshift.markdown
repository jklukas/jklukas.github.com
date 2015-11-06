---
layout: post
title: "Safe Migrations with Redshift"
date: 2015-04-07 10:54:12 -0500
comments: true
categories:
---

*This post originally appeared on the [Simple Engineering Blog](
https://www.simple.com/engineering/safe-migrations-with-redshift).*

Simple has adopted [Amazon Redshift](https://aws.amazon.com/redshift/)
as a data warehouse. It's used not just by our data analysts, but by
teams throughout the company to answer questions about
fraud, customer behavior, and more.

<!-- more -->

### Simple's Analytics Infrastructure

A year ago, many business intelligence questions at Simple
could only be answered via direct access to production databases.
Aside from the difficulty that presented from a security and
access standpoint, the inability to aggregate information
from disparate data sources
tied our hands in terms of the depth of insights we could achieve.

The need for a data warehouse was clear.
We chose Redshift as our warehouse for ease of use, scalability,
and cost. It also integrates well with our existing platform,
which runs largely on AWS.

We have built our pipelines to Redshift such that
updates are pushed in real time rather than pulled nightly
as in a traditional ETL (extract, transform, load) process.
As such, our warehouse is never static and
[Redshift’s documented approaches to deep copies](https://docs.aws.amazon.com/redshift/latest/dg/performing-a-deep-copy.html)
needed some customization.
In order to retain updates while recreating tables,
we had to dive deep into the documentation, do some experimentation,
and finally make some compromises in terms of warehouse availability.

### The Need for Deep Copies

Although Redshift presents a client interface that mimics PostgreSQL,
operational details make the experience of managing a Redshift cluster
very different from a traditional SQL database.
One management struggle that we've dealt with is the need to perform
deep copies of tables for certain structural changes.
Redshift supports `ALTER TABLE`
statements to change ownership and add/drop columns,
but other important changes require tearing the table down and starting over.

As we have gained operational experience both with Redshift and
and with our own data usage patterns,
we have in several cases needed to change
[distribution styles](https://docs.aws.amazon.com/redshift/latest/dg/c_best-practices-best-dist-key.html)
which determine where data lives within a cluster and
[sort keys](https://docs.aws.amazon.com/redshift/latest/dg/c_best-practices-sort-key.html)
which are akin to database indexes.
Because these two properties cannot be altered, we have had to learn
how to efficiently copy data to a new table and swap it with the original.

### Can Schema Changes and Transactions Mix?

Simple's backend engineering team uses [Flyway](http://flywaydb.org/)
to perform migrations on PostgreSQL databases, and our data engineers
have followed suit for maintaining our schema in Redshift.
Flyway automatically wraps every migration in a transaction
so that when a single statement fails, the entire migration can be rolled back.
As discussed in the
[Flyway FAQ](http://flywaydb.org/documentation/faq.html#rollback),
however, not all databases agree about how to handle DDL statements
(like creating, dropping, and renaming tables) within transactions:

> Today only DB2, PostgreSQL, Derby and to a certain extent SQL Server support DDL statements inside a transaction. Other databases such as Oracle will implicitly sneak in a commit before and after each DDL statement, drastically reducing the effectiveness of this roll back. One alternative if you want to work around this, is to include only a single DDL statement per migration. This solution however has the drawback of being quite cumbersome.

Redshift doesn't wrap DDL statements with implicit commits,
so rolling back a failed transaction *is* possible
even when it has made changes to the database schema.
But how do concurrent transactions behave
while these schema changes are in progress?
Amazon’s documentation on
[serializable isolation in Redshift](https://docs.aws.amazon.com/redshift/latest/dg/c_serial_isolation.html)
discusses how write locks are used during concurrent transactions to protect
data integrity and gives us a hint about how DDL statements are handled:

> System catalog tables (PG) and other Amazon Redshift system tables (STL and STV) are not locked in a transaction; therefore, changes to database objects that arise from DDL and TRUNCATE operations are visible on commit to any concurrent transactions.

By rolling up our sleeves and experimenting with the sequencing
of statements within a migration, we were able
to develop a more concrete understanding of how write locks and DDL
operations are applied.

### Safely Recreating a Table

Let's discuss two examples using the “`tickit`” database from Redshift's
[Getting Started Guide](https://docs.aws.amazon.com/redshift/latest/dg/c_intro_to_admin.html).

Here's a great way to lose data during a migration (remember that Flyway
automatically wraps this in a transaction):

```sql
-- potential data loss
-- you probably don't want to do this
CREATE TABLE sales_copy AS (SELECT * FROM sales);
DROP TABLE sales;
ALTER TABLE sales_copy RENAME TO sales;
```

During the creation of `sales_copy`, the original `sales` table is fully
available to other processes for reads and writes.
As soon as the `DROP TABLE` statement is executed, however, Redshift places
a lock on `sales` and all new queries that touch the `sales` table will
block until the transaction finishes.
If either the `DROP TABLE` or `ALTER TABLE` commands fail,
the transaction is rolled back cleanly since the DDL changes
were never made visible to any other processes.

The above migration is only dangerous if it succeeds:
when the transaction commits,
the original table is dropped along with all updates.
By the time the read/write lock is put in place,
other processes may have already made updates to `sales`.
Because those updates aren't visible within our transaction,
they never make it into `sales_copy`.

Our solution was to change the order of operations so that
the first line of the migration is a DDL statement affecting `sales`,
ensuring that reads and writes to that table are blocked immediately.
The lock persists all the way through transaction commit,
when the transaction’s schema changes are finally made visible to other processes.
Placing that lock at the beginning of the transaction means
a longer period during which the table is unavailable for queries,
but we judge that a small price to pay for integrity of the warehouse.
To minimize the duration of the outage,
we follow the advice of the Flyway maintainers and modify
only a single table per migration.

Our table recreation strategy now looks like this:

```sql
-- a safer sequencing of events
ALTER TABLE sales RENAME TO sales_old;
CREATE TABLE sales AS (SELECT * FROM sales_old);
DROP TABLE sales_old;
```

### Tooling for Redshift

Three lines of SQL is fairly easy to write by hand and makes for a
perfectly sufficient migration when you simply need to clean up
a table with too many columns to support a `VACUUM` operation.
The process becomes significantly more complex when you want to
change distribution or sort keys.

We are still building out our toolset to make
performing these sorts of maintenance tasks easier, and we look
forward to providing many of those tools to the community as open
source projects. We recently released
[Pipewelder](https://github.com/SimpleFinance/pipewelder),
a framework for defining scheduled tasks under version control
and deploying them via [AWS Data Pipeline](https://aws.amazon.com/datapipeline/).
We have deployed Pipewelder both to handle nightly `VACUUM` operations
and to create hourly derived tables, which we use to mimic materialized views.
