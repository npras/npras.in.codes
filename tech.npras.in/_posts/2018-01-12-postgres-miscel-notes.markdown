---
layout: post
title: "PostgreSQL notes: Miscellaneous"
excerpt: "Everything else from the MasteringPostgreSQL book"
---

I just finished reading the excellent [__Mastering PostgreSQL__](https://masteringpostgresql.com/) by Dimitri Fontaine. These are the left out notes about various topics from the book.

I'm in no way a _master_ at this, but I've begun to explore SQL in depth. I want to be able to think easily in terms of relations, and joins. There are sites like [PGExercises](https://pgexercises.com/) and [LeetCode](https://leetcode.com/) that I can use to practice. I also would like to be able to practice modeling the database for any application. In the book, Dimitri did one such example where he designed the schema and tested out the queries for a few mostly commonly expected scenarios. I like that approach.

Anyway, here are the notes.

PHP/Ruby/Javascript is an object-oriented imperative programming language which means it is good at execution control logic.
SQL is a set-oriented declarative programming language and is perfect for data computing.

The support of unstructured types like XML or JSON in relational databases is a huge step forward in focusing on what’s really important. Now, in one field there can be labels with translation, multiple postal addresses, media definitions, etc. that were creating a lot of noise in the database schemas before. These are application-oriented structures. It means the database does not have to care about their consistency and they are complex business structure for the application layer.

You know your database schema is good when all the very simple business cases turn out to be implemented as rather simple SQL queries, yet it’s still possible to address very specific advanced needs in reporting or fraud detection, or accounting oddities.

There’s a fundamental difference between the application’s design of its internal state (object-oriented or not) and the database model, though:

* The application implements workflows, user stories, ways to interact with the system with presentation layers, input systems, event collection APIs and other dynamic and user-oriented activities.
* The database model ensures a consistent view of the whole world at all times, allowing every actor to mind their own business and protecting them from each other so that the world you are working with continues to make sense as a whole.


__Database anamolies:__ insert/update/delete types. To avoid these, normalize your db.


__database vs schema:__ A PostgreSQL "schema" is roughly the same as a MySQL "database". A schema contains a group of tables. A database contains a group of schemas. Usecase of schema: u can have all tables in 1 particular schema X, and then in another schema Y, you can have only views of tables from X, and then in another schema Z, you can have only materialized views. You could grant permissions at schema level thereby restricting access to the actual tables.


#### Import a SQL file into a database
* create db first, via pgadmin
* from terminal: psql dbname < sqlfile.sql


#### replace a string
```sql
select replace('prasanna', 'anna', 'bro');
select regexp_replace('foo bar boz barbaz', 'Bar', 'baz', 'gi')
select substring(replace('$1,000,999', ',', '') from 2); -- 1000999
```


#### like
```sql
select 'abc' like 'a%'; -- true
select 'abc' like '_b_'; -- true
```


#### typecasting
```sql
select date '2017-02-01';
select '2017-02-01'::date;
select cast ('2017-02-01' as DATE);
```


#### generate_series
```sql
-- displays all days of Feb from start to end
-- look at how the dates are typecast and intervals too
select generate_series(date '2017-02-01', 
                       date '2017-02-01' + interval '1 month' - interval '1 day', 
                       interval '1 day');
 ```


#### nullif function
The NULLIF function returns a null value if argument_1 equals to argument_2, otherwise it returns argument_1.
```sql
select NULLIF(1, 1); -- NULL
select NULLIF(0, 1); -- 0
select NULLIF('A', 'B'); -- 'A'
```


#### coalesce function
returns the first non-null argument passed to it. any number of args can be passed.

eg: if excerpt is present, select it, otherwise select first 40 chars of body:

`select title, coalesce(nullif(excerpt, ''), left(body, 7)) as summary from posts;`


#### CTEs - Common Table Expressions
Used in a single transaction. The CTE creates a computed table for use just in that query.

```sql
WITH computed_table_name as 
(
 -- complex SELECT or other queries
)

select x, y from computed_table_name;
```


#### quotes in sql query
Double quotes for table and column names. 
Single quotes for string literals in where clause. 
For a literal with single quote, escape it with another single quote. 
Eg: `select * from employees where name = 'John''s child'`


#### group by
The GROUP BY Clause is used to group together those rows in a table that have the same values in all the columns listed.
In general, if a table is grouped, columns that are not listed in GROUP BY cannot be referenced except in aggregate expressions. 

group by with multiple cols explained with example here: https://stackoverflow.com/questions/2421388/using-group-by-on-multiple-columns


#### JOIN and WHERE
Some results can be obtained by 2 ways: using joins or just where.

```sql
select
    artist."Name" artistName,
    album."Title" albumTitle,
    track."Name" trackName
  from "Artist" artist
    join "Album" album using("ArtistId")
    join "Track" track using("AlbumId")
  where album."AlbumId" = 193
;

-- using just where
select
    artist."Name" artistName,
    album."Title" albumTitle,
    track."Name" trackName
  from "Artist" artist, "Album" album, "Track" track
  where
    album."AlbumId" = 193 and
    artist."ArtistId" = album."ArtistId" and
    album."AlbumId" = track."AlbumId"
;
```


#### Boolean output
Boolean columns in SELECT: -- returns 't' for true and 'f' for false
```psql
~# select 1 = 1;
 ?column?
══════════
 t
(1 row)

~# select 1 = 2;
 ?column?
══════════
 f
(1 row)
```


#### How "Order by multiple columns work?"
```sql
SELECT * FROM Customers
ORDER BY Country DESC,CustomerName;
```

It orders by Country, but if some rows have the same Country, it orders them by CustomerName.


#### nulls last
"order by position nulls last"

Complex `Order by`: with nulls last, multiple columns, `case when`: List all drivers of a specific race with the winners at top. And if 2 drivers hold same position, show the driver who finished the most laps higher. And also if 2 drivers have same position and lap counts, show the driver with the status 'Power Unit' higher.

Expected output:

```
 code │  surname   │ position │ laps │   status
══════╪════════════╪══════════╪══════╪════════════
 BOT  │ Bottas     │        1 │   52 │ Finished
 VET  │ Vettel     │        2 │   52 │ Finished
 RAI  │ Räikkönen  │        3 │   52 │ Finished
 HAM  │ Hamilton   │        4 │   52 │ Finished
 VER  │ Verstappen │        5 │   52 │ Finished
 PER  │ Pérez      │        6 │   52 │ Finished
 OCO  │ Ocon       │        7 │   52 │ Finished
 HUL  │ Hülkenberg │        8 │   52 │ Finished
 MAS  │ Massa      │        9 │   51 │ +1 Lap
 SAI  │ Sainz      │       10 │   51 │ +1 Lap
 STR  │ Stroll     │       11 │   51 │ +1 Lap
 KVY  │ Kvyat      │       12 │   51 │ +1 Lap
 MAG  │ Magnussen  │       13 │   51 │ +1 Lap
 VAN  │ Vandoorne  │       14 │   51 │ +1 Lap
 ERI  │ Ericsson   │       15 │   51 │ +1 Lap
 WEH  │ Wehrlein   │       16 │   50 │ +2 Laps
 RIC  │ Ricciardo  │        ¤ │    5 │ Brakes
 ALO  │ Alonso     │        ¤ │    0 │ Power Unit
 PAL  │ Palmer     │        ¤ │    0 │ Collision
 GRO  │ Grosjean   │        ¤ │    0 │ Collision
```

And the sql is:

```sql
select
    drivers.code,
    drivers.surname,
    position,
    laps,
    status
  from results
    join drivers using(driverid)
    join status using(statusid)
  where raceid = 972
  order by
    position nulls last,
    laps desc,
    case when status = 'Power Unit'
      then 1 -- if ASC, then give any lowest integer compared to below
      else 2
    end
;
```


#### Indexes
PostgreSQL provides several index types: B-tree (most common), Hash, GiST, SP-GiST, GIN and BRIN.

By default, B-tree indexes store their entries in ascending order with nulls last. This means that a forward scan of an index on column x produces output satisfying ORDER BY x (or more verbosely, ORDER BY x ASC NULLS LAST). The index can also be scanned backward, producing output satisfying ORDER BY x DESC (or more verbosely, ORDER BY x DESC NULLS FIRST, since NULLS FIRST is the default for ORDER BY DESC).

You can adjust the ordering of a B-tree index by including the options ASC, DESC, NULLS FIRST, and/or NULLS LAST when creating the index; for example:

```sql
CREATE INDEX test2_info_nulls_low ON test2 (info NULLS FIRST);
CREATE INDEX test3_desc_index ON test3 (id DESC NULLS LAST);
```

You might wonder why bother providing all four options, when two options together with the possibility of backward scan would cover all the variants of ORDER BY. In single-column indexes the options are indeed redundant, but in multicolumn indexes they can be useful. Consider a two-column index on (x, y): this can satisfy `ORDER BY x, y` if we scan forward, or `ORDER BY x DESC, y DESC` if we scan backward. But it might be that the application frequently needs to use `ORDER BY x ASC, y DESC`. There is no way to get that ordering from a plain index, but it is possible if the index is defined as `(x ASC, y DESC) or (x DESC, y ASC)`.

__Indexes on Expressions:__ useful if more queries are computed:
```sql
CREATE INDEX test1_lower_col1_idx ON test1 (lower(col1));
```
for query:
```sql
SELECT * FROM test1 WHERE lower(col1) = 'value';
```

and this index:
```sql
CREATE INDEX people_names ON people ((first_name || ' ' || last_name));
```

for this query:
```sql
SELECT * FROM people WHERE (first_name || ' ' || last_name) = 'John Smith';
```


#### point datatype
(x, y) and distance-between operator <->

Eg: select 10 circuits closer to a point:
```sql
select name, location, country from circuits order by point(lng,lat) <-> point(2.349014, 48.864716) limit 10;
```


create gist index for point types:
```sql
alter table f1db.circuits add column position point;
update f1db.circuits set position = point(lng,lat);
create index on f1db.circuits using gist(position);
```


#### UNION vs UNION ALL
UNION removes duplicate records (where all columns in the results are the same), UNION ALL does not.

```sql
SELECT 'foo' AS bar UNION SELECT 'foo' AS bar
SELECT 'foo' AS bar UNION ALL SELECT 'foo' AS bar
```

---

`LIMIT 3` in a query can be written using SQL standard format as `fetch first 3 rows only`

---


#### Pagination without OFFSET
Don't use OFFSET for pagination. For farther pages, all of the offset'ed records will still be scanned.
Instead do like this:

For the first 3 records:
```sql
select
    lap, drivers.code, position, milliseconds * interval '1ms' as laptime
  from laptimes
    join drivers using(driverid)
  where raceid = 972
  order by lap, position
  fetch first 3 rows only
  -- same as above
  -- limit 3
;
```

Now, the app should make an effort to extract and save the info that the last uniq (lap, position) combo was (1, 3) as it is used in next subsequent pagination query:
```sql
select
    lap, drivers.code, position, milliseconds * interval '1ms' as laptime
  from laptimes
    join drivers using(driverid)
  where raceid = 972
    and row(lap, position) > (1, 3)
  order by lap, position
  fetch first 3 rows only
  -- same as above + the row() condition
  -- limit 3 offset 3
;
```


#### GROUP BY is map-reduce!
map your data into different groups, and in each group reduce the data set to a single value.

First map the results in a way you want, and then reduce the groups, maybe using aggregate fns.

Eg: list number of races in each decade
Map first:
```sql
select
    extract('year' from date_trunc('decade', date)) as decade
  from races
  order by decade
;
```

This just lists 1 row for each race formatted as the coresponding decade year.
And then, using group by, we could reduce the group records using some aggregate fn:
```sql
select
    extract('year' from date_trunc('decade', date)) as decade,
    count(*)
  from races
  group by decade
  order by decade
;
```

And the result:
```
 decade │ count
════════╪═══════
   1950 │    84
   1960 │   100
   1970 │   144
```


#### aggregate with `filter` clause and `bool_and`

`filter` clause with aggregate fns.
`select count(*) filter(where position is null) from X`

It means only count rows where position is null.

Eg: list total count, evens count and fives count from 1 to 50.
Expected result:
```
total │ evens │ fives
═══════╪═══════╪═══════
    50 │    25 │    10
```

And the query is:
```sql
select
    count(*) total,
    count(*) filter(where x %2 = 0) evens,
    count(*) filter(where x %5 = 0) fives
  from generate_series(1, 50) t(x)
;
```

Eg: list top 5 years where most players didn't finish the race.

Expected result:
```
 season │ #times any driver din't finish a race
════════╪═══════════════════════════════════════
   1989 │                                   139
   1953 │                                    51
   1955 │                                    48
   1990 │                                    48
   1956 │                                    46
```
And the query is:
```sql
with counts as
(
  select
      date_trunc('year', date) as year,
      count(*) participated,
      count(*) filter(where position is null) outs,
      bool_and(position is null) never_finished
    from drivers
      join results using(driverid)
      join races using(raceid)
    group by date_trunc('year', date), driverid
)

-- select * from counts;

select
    extract(year from year) as season,
    sum(outs) as "#times any driver din't finish a race"
  from counts
  where never_finished
  group by season
  order by sum(outs) desc
  limit 5;
```

`counts` CTE has these counts:
* participated: count of all participations by a driver in a year
* outs: count of all races by a driver when he didn't finish

`bool_and(expression)`. Used to return `t` or `f` depending on expr. If expression is satisfied for all records, then `t`, else `f`.


#### Grouping sets
More complex grouping operations than those described above are possible using the concept of grouping sets. 
The data selected by the FROM and WHERE clauses is grouped separately by each specified grouping set, aggregates computed for each group just as for simple GROUP BY clauses, and then the results returned

```
=> SELECT * FROM items_sold;
 brand | size | sales
-------+------+-------
 Foo   | L    |  10
 Foo   | M    |  20
 Bar   | M    |  15
 Bar   | L    |  5
(4 rows)

=> SELECT brand, size, sum(sales) FROM items_sold GROUP BY GROUPING SETS ((brand), (size), ());
 brand | size | sum
-------+------+-----
 Foo   |      |  30
 Bar   |      |  20
       | L    |  15
       | M    |  35
       |      |  50
(5 rows)
```


#### DISTINCT ON vs GROUP BY

SELECT DISTINCT ON ( expression [, . . . ] ) keeps only the first row of each set of rows where the given expressions evaluate to equal.

Eg: list of all drivers who have ever won a race.

Expected output:
```
   forename   │    surname
══════════════╪═══════════════
 Lewis        │ Hamilton
 Nico         │ Rosberg
 Fernando     │ Alonso
 Heikki       │ Kovalainen
```

The sql is:
```sql
select
    distinct on (driverid)
    forename,
    surname
from results
join drivers using(driverid)
where position = 1
;
```

Note that this could also be written using group by like this:
```sql
select
    forename,
    surname
from results
join drivers using(driverid)
where position = 1
group by drivers.driverid
;
```


#### nulls

Suppose we have a cross-joined table with true, false and null like this:
```
 a │ b
═══╪═══
 t │ t
 t │ f
 t │ ¤
 f │ t
 f │ f
 f │ ¤
 ¤ │ t
 ¤ │ f
 ¤ │ ¤
```

which is created using this sql:
```sql
select
    a,
    b
  from (values(true), (false), (null)) v1(a)
    cross join (values(true), (false), (null)) v2(b)
;
```

Never use `a = NULL`. Instead use `a IS NULL` in `where` clause.

```sql
select
    a,
    b
  from (values(true), (false), (null)) v1(a)
    cross join (values(true), (false), (null)) v2(b)
  where a = null
;
```
The above query produces no results. 0 rows.

But,
```sql
select
    a,
    b
  from (values(true), (false), (null)) v1(a)
    cross join (values(true), (false), (null)) v2(b)
  where a is null
;
```

produces the expected result:
```
 a │ b
═══╪═══
 ¤ │ t
 ¤ │ f
 ¤ │ ¤
```


#### regex

`select 'thomas' ~ '.*thomas.*';` returns true

`'thomas' ~ '.*thomas.*'` - case sensitive
`'thomas' ~* '.*Thomas.*'` - case insensitive
`'thomas' !~ '.*Thomas.*'` - case sensitive
`'thomas' !~* '.*vadim.*'` - case insensitive

`regexp_replace('foobarbaz', 'b..', 'X')` fooXbaz
`regexp_replace('foobarbaz', 'b..', 'X', 'g')` fooXX

```
SELECT regexp_match('foobarbequebaz', 'bar.*que');
 regexp_match
--------------
 {barbeque}
(1 row)

SELECT regexp_match('foobarbequebaz', '(bar)(beque)');
 regexp_match
--------------
 {bar,beque}
(1 row)
```


#### range types
PostgreSQL comes with the following built-in range types:
    int4range — Range of integer
    int8range — Range of bigint
    numrange — Range of numeric
    tsrange — Range of timestamp without time zone
    tstzrange — Range of timestamp with time zone
    daterange — Range of date


Range examples:
```sql
CREATE TABLE reservation (room int, during tsrange);
INSERT INTO reservation VALUES
    (1108, '[2010-01-01 14:30, 2010-01-01 15:30)');

-- Containment
SELECT int4range(10, 20) @> 3; -- false

-- Overlaps
SELECT numrange(11.1, 22.2) && numrange(20.0, 30.0); -- true

-- Extract the upper bound
SELECT upper(int8range(15, 25)); -- 25

-- Compute the intersection
SELECT int4range(10, 20) * int4range(15, 25); -- [15,20)

-- Is the range empty?
SELECT isempty(numrange(1, 5)); -- false
```


#### Array datatype
Create a GIN index for this. It indexes the contents of the array as opposed to the array as a whole.

the `unnest` function takes an array as its arg and returns a relation.
```
f1db# select array['#job', '#blah'];
    array
══════════════
 {#job,#blah}
(1 row)

f1db# select unnest(array['#job', '#blah']);
 unnest
════════
 #job
 #blah
```

Array contains operator: `@>`:
```
f1db# select array['#job', '#blah'] @> array['#blah'];
 ?column?
══════════
 t
(1 row)
```


#### JSON datatype
`json` is different from `jsonb` wrt data types.
jsonb means better json.

Use jsonb. It is an advanced binary storage format with full processing, indexing and searching capabilities, and as such pre- processes the JSON data to an internal format, which does include a single value per key; and also isn’t sensible to extra whitespace or indentation.

jsonb `containment @>` operator:
```
select * from js where extra @> '2';
 id │    extra
════╪══════════════
1   │ [1, 2, 3, 4]
  2 │ [2, 3, 5, 8]
(2 rows)

-- And json array containing another array:
select * from js where extra @> '[2,4]';
 id │    extra
════╪══════════════
  1 │ [1, 2, 3, 4]
(1 row)
```

jsonb `existence` operator `?`:
jsonb also has an existence operator, which is a variation on the theme of containment: it tests whether a string (given as a text value) appears as an object key or array element at the top level of the jsonb value.
```
-- String exists as array element:
SELECT '["foo", "bar", "baz"]'::jsonb ? 'bar';

-- String exists as object key:
SELECT '{"foo": "bar"}'::jsonb ? 'foo';

-- Object values are not considered:
SELECT '{"foo": "bar"}'::jsonb ? 'bar';  -- yields false
```


#### hstore data type (available as extension)
The text representation of an hstore, used for input and output, includes zero or more key => value pairs separated by commas:
```
k => v
foo => bar, baz => whatever
"1-a" => "anything at all"
```


---

Given a table with a single column of words, write a query to output a string containing a random 3 words concatenated.

Expected output:

```
        string_agg
══════════════════════════
 elit aliqua consequuntur
 ```

 And the sql is:

```sql
  with words(w) as (
  select word
  from sandbox.lorem
  order by random() limit 3
  )
  select string_agg(w, ' ')
  from words;
```

#### Views

Views are virtual tables. No data is persisted separately from the original tables.
Materialized views are stored physically like a separate table. After creating the m.view, you could populate/repopulate it with fresh data with the `REFRESH MATERIALIZED VIEW view_name;` command.

Can use cron to update this m.view and use the data as a cache for some dashboard data.


#### Enum type

Can be created using `CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');`
and used in table creation as yet another type: 
`CREATE TABLE person (
    name text,
    current_mood mood
);`



#### recursive WITH
From the docs:

The general form of a recursive WITH query is always a non-recursive term, then UNION (or UNION ALL), then a recursive term, where only the recursive term can contain a reference to the query's own output.

Recursive queries are typically used to deal with hierarchical or tree-structured data. An example is [here](https://tech.npras.in/postgres-recursive-query/).

Query to get sum of all numbers from 1 to 100:
```sql
with recursive t(n) as (
  values (1)
  union all
  select n+1 from t where n < 100
)

select sum(n) from t
;
```


#### Data-Modifying Statements in WITH
This query effectively moves rows from products to products_log. The DELETE in WITH deletes the specified rows from products, returning their contents by means of its RETURNING clause; and then the primary query reads that output and inserts it into products_log.
```sql
WITH moved_rows AS (
    DELETE FROM products
    WHERE
        "date" >= '2010-10-01' AND
        "date" < '2010-11-01'
    RETURNING *
)
INSERT INTO products_log
SELECT * FROM moved_rows;
```


#### `RETURNING` with insert, update, delete

After perforning these operations, we can tell the query to return the values of the row. Useful as it avoids extra query.

Eg: when deleting multiple rows, use returning to return a summary of what was deleted:

```sql
with deleted_rows as
(
  delete
    from tweet.users
  where not exists
  (
    select 1 from tweet.message where userid = users.id
  )
  returning *
)
select min(userid), max(userid), count(*), array_agg(uname) from deleted_rows
;
```

It gives a nice summary of deleted rows.


#### bulk update and bulk assertions

```sql
  update mom.artist
    set (col1, col2, col3)
        = (batch.col1, batch.col2, batch.col3)
    from batch
    where batch.this_id = artist.this_id
    and
    (artist.col1, artist.col2, artist.col3)
    <> (batch.col1, batch.col2, batch.col3)
;
```


#### psql commands
* set and use variables:
`\set start '2017-02-01'`
`select :'start';`

* connect to a db:
`\c dbname`

* list all tables in current db:
`\dt`

* describe a table:
`\d table_name;`

* run queries from a sql file:
`\i path-to-file.sql

* from terminal, set variables and run queries from a sql file:
`$psql --variable "n=10" -f artist.sql chinook`
