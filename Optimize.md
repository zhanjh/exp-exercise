# Optimize

## nodejs

### Don't use synchronous functions

* Trace the usage of synchronous API by command-line flag `--trace-sync-io`

### Logging & Debugging

* console.log or console.error are not recommended, since they are synchronous.
* Debugging module: [debug](https://www.npmjs.com/package/debug)
* Logging module: [winston](https://www.npmjs.com/package/winston)

### Handle exceptions properly

* Should not to listen for the uncaughtException event.
* Use try-catch
* Use promises

### Things to do in  your environment / setup

* Set NODE_ENV to "production"
* Use a process manager to ensoure app restarts if it crashes.
	* PM2 / Forever / StrongLoop Process Manager
* Use an init system
	* Systemd
* Run you app in a cluster
* Use a load balancer
* Use a reverse proxy

## Database

### Performance issue caused by View `ContactSummary`

The MERGE algorithm cannot be used because of `GROUP BY`. See more about [MERGE Limitations](https://mariadb.com/kb/en/library/view-algorithms/#merge-limitations)

With View

```sql
explain SELECT UserID, Title, Name, BirthDate, IsFavorite, DetailCount FROM ContactSummary  LIMIT 0, 10 \G;

***************************[ 1. row ]***************************
id            | 1
select_type   | PRIMARY
table         | <derived2>
type          | ALL
possible_keys | <null>
key           | <null>
key_len       | <null>
ref           | <null>
rows          | 20508
Extra         |
***************************[ 2. row ]***************************
id            | 2
select_type   | DERIVED
table         | c
type          | ALL
possible_keys | <null>
key           | <null>
key_len       | <null>
ref           | <null>
rows          | 10254
Extra         | Using temporary; Using filesort
***************************[ 3. row ]***************************
id            | 2
select_type   | DERIVED
table         | d
type          | ref
possible_keys | UserID
key           | UserID
key_len       | 4
ref           | Expedia.c.UserID
rows          | 2
Extra         | Using index
```

Without View

```
explain select c.`UserID`, c.`Title`, c.Name, c.BirthDate, c.IsFavorite, count(d.`UserID`) as DetailCount
from `Contact` `c` join `ContactDetail` `d` using(`UserID`) group by `d`.`UserID` limit 0, 10 \G;

*************************** [1. row ]***************************
id            | 1
select_type   | SIMPLE
table         | c
type          | index
possible_keys | PRIMARY
key           | PRIMARY
key_len       | 4
ref           | <null>
rows          | 5
Extra         |
***************************[ 2. row ]***************************
id            | 1
select_type   | SIMPLE
table         | d
type          | ref
possible_keys | UserID
key           | UserID
key_len       | 4
ref           | Expedia.c.UserID
rows          | 2
Extra         |
```

### Add index for ORDER BY

```
alter table `Contact` add key `Name` (`Name`);
```

Before

```sql
explain select * from `Contact` order by `Name` limit 0, 10 \G;

***************************[ 1. row ]***************************
id            | 1
select_type   | SIMPLE
table         | Contact
type          | ALL
possible_keys | <null>
key           | <null>
key_len       | <null>
ref           | <null>
rows          | 10254
Extra         | Using filesort
```

After
```sql
***************************[ 1. row ]***************************
id            | 1
select_type   | SIMPLE
table         | Contact
type          | index
possible_keys | <null>
key           | Name
key_len       | 602
ref           | <null>
rows          | 10
Extra         |
```

### Full-Text Index Overview

* <https://mariadb.com/kb/en/library/full-text-index-overview/>

## ref
* [Production best practices: performance and reliability](https://expressjs.com/en/advanced/best-practice-performance.html#dont-use-synchronous-functions)
