---
layout: post
title: "PostgreSQL notes: Denormalization techniques"
excerpt: "Being fanatic about being normalized doesn't help the fanatics"
---


I'm currently reading the excellent [__Mastering PostgreSQL__](https://masteringpostgresql.com/) by Dimitri Fontaine. It explained a few denormalization techniques to keep in mind when designing a database.

Normalized DB will have many tables and a lot of foreign keys connecting them. Getting proper data means having to join multiple tables.

These might be considered too heavy in some cases and the queries might be too slow even with index.

It's better to denormalize in these cases.

Some techniques include:

### Materialized views
Create once and refresh as needed to store data collected from heavy queries.


### History tables and audit trials
To track insert/delete/update on rows of all tables, use a single table that stores the data as json or hstore.

```sql
create schema if not exists archive;

create type archive.row_action as enum('insert', 'delete', 'update');

create table archive.older_versions(
table_name text,
date timestamptz default now(),
action archive.action_t,
data jsonb
);

-- inserting new record will look like this:
insert into archive.older_versions(table_name, action, data)
    select 'hashtag', 'delete', row_to_json(hashtag)
    from hashtag
    where id = 23838
    returning select 'hashtag', 'delete', row_to_json(hashtag);
```


### range types for validity periods
Eg: gold rate or dollar rates change over dates. daterange types can be used to store them.


### pre-computed values
In some cases, the application keeps computing the same derived values each time it accesses to the data. It’s easy to pre-compute the value with PostgreSQL:
• As a default value for the column if the computation rules only in- clude information available in the same tuple
• With a before trigger that computes the value and stores it into a column right in your table


### Enum data types
It is possible to use ENUM rather than a reference table.

When dealing with a short list of items, the normalized way to do that is to handle the catalog of accepted values in a dedicated table and reference this table everywhere your schema uses that catalog of values.
When using more than join_collapse_limit or from_collapse_limit rela- tions in SQL queries, the PostgreSQL optimizer might be defeated. . . so in some schema using an ENUM data type rather than a reference table can be beneficial.


### Multiple values per attribute
Suppose each user has a lot of preference values to store. Instead of separate columns for each, store them all in a composite type like array (list of entries) or jsonb (preferences).

### table partitioning
https://www.postgresql.org/docs/10/static/ddl-partitioning.html
