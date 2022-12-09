---
title: Databudgie
template: projects
link: https://databudgie.readthedocs.io/en/latest/index.html
date: 2019-10-10
weight: 3
---

Databudgie is a CLI & library for database performing targeted backup and restore of database tables or arbitrary queries against database tables.

Some minimal config might look like:

```yaml
# config.databudgie.yml
backup:
  url: postgresql://postgres:postgres@localhost:5432/postgres

restore:
  url: postgresql://postgres:postgres@localhost:5432/postgres

tables:
  - some_schema.*
  - author
  - book
  - name: chapter
    query: "select * from {table} where book_id > 4"
  - name: some_table
    follow_foreign_keys: true
```

Think `pg_dump`, but more flexible and easy to produce backups for a targeted
set of tables/data, and restore them.
