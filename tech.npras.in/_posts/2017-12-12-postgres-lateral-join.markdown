---
layout: post
title: "PostgreSQL notes: Use lateral joins to run loops in a single query"
excerpt: "For each row returned by the outer query, the inner query will be run"
---

I'm currently reading the excellent [__Mastering PostgreSQL__](https://masteringpostgresql.com/) by Dimitri Fontaine. Came across this nice ability to perform `lateral join`.

Loosely, it means that a LATERAL join is like a SQL foreach loop, in which PostgreSQL will iterate over each row in a result set and evaluate a subquery using that row as a parameter.

The subquery specified on the RIGHT SIDE of the JOIN is evaluated for each row on the LEFT side of the JOIN.


#### Basic example
Using fake table data generator:

```sql
WITH foo AS
(
  SELECT
    'a' as name, 2 quantity
  UNION ALL
  SELECT
    'b' as name, 4 quantity
)

select * from foo;
```

gives us this table:

```
 name │ quantity
══════╪══════════
 a    │        2
 b    │        4
 ```
 
Now, using lateral join, we could loop like this:

```sql
WITH foo AS
(
  SELECT
    'a' as name, 2 quantity
  UNION ALL
  SELECT
    'b' as name, 4 quantity
)

-- select * from foo;

SELECT
    t.name,
    t.quantity,
    x
  FROM foo t,
-- No need to say "LATERAL" here as it's added automatically
    LATERAL
    generate_series(1, quantity) x
;
```

which produces this:
```
 name │ quantity │ x
══════╪══════════╪═══
 a    │        2 │ 1
 a    │        2 │ 2
 b    │        4 │ 1
 b    │        4 │ 2
 b    │        4 │ 3
 b    │        4 │ 4
 ```
 
Another usecase: __Top-N per group__

Suppose you want __top 3 winners of all races conducted in 2009__. We could use lateral join.

The outer query will filter out races based on year as 2009 and will list all races during 2009.
And the subquery inside the lateral join will find top 3 winners for each of the race listed from outer query.

For this to work, the subquery will refer the raceid from the outer query. Lateral join specifically allows this.
 
 ```sql
select
    races.year,
    races.raceid,
    races.name,
    lat.surname,  -- note the tableref. It comes from the join
    lat.position
  from races
    left join lateral
    (
      select
          drivers.surname,
          results.position
        from results
        join drivers using(driverid)
        where results.raceid = races.raceid -- note this. It comes from outer query
        order by position nulls last
        limit 3
    ) lat on true
  where races.year = 2009
;
```

and the result is:
```
 year │ raceid │         name          │   surname   │ position
══════╪════════╪═══════════════════════╪═════════════╪══════════
 2009 │      1 │ Australian Grand Prix │ Button      │        1
 2009 │      1 │ Australian Grand Prix │ Barrichello │        2
 2009 │      1 │ Australian Grand Prix │ Trulli      │        3
 2009 │      2 │ Malaysian Grand Prix  │ Button      │        1
 2009 │      2 │ Malaysian Grand Prix  │ Heidfeld    │        2
 2009 │      2 │ Malaysian Grand Prix  │ Glock       │        3
 2009 │      3 │ Chinese Grand Prix    │ Vettel      │        1
 2009 │      3 │ Chinese Grand Prix    │ Webber      │        2
 2009 │      3 │ Chinese Grand Prix    │ Button      │        3
 2009 │      4 │ Bahrain Grand Prix    │ Button      │        1
 2009 │      4 │ Bahrain Grand Prix    │ Vettel      │        2
 2009 │      4 │ Bahrain Grand Prix    │ Trulli      │        3
 2009 │      5 │ Spanish Grand Prix    │ Button      │        1
 2009 │      5 │ Spanish Grand Prix    │ Barrichello │        2
 2009 │      5 │ Spanish Grand Prix    │ Webber      │        3
 2009 │      6 │ Monaco Grand Prix     │ Button      │        1
 2009 │      6 │ Monaco Grand Prix     │ Barrichello │        2
 2009 │      6 │ Monaco Grand Prix     │ Räikkönen   │        3
 2009 │      7 │ Turkish Grand Prix    │ Button      │        1
 2009 │      7 │ Turkish Grand Prix    │ Webber      │        2
 2009 │      7 │ Turkish Grand Prix    │ Vettel      │        3
```

Another similar scenario: __list of the top three drivers in terms of races won, by decade__.

```sql
with decades as
(
  select
      distinct extract('year' from date_trunc('decade', races.date)) decade
    from races
)

-- select * from decades;

select
    decades.decade,
    lat.surname,
    rank() over(partition by decades.decade order by lat.wins desc),
    lat.wins
  from decades
    left join lateral
    (
      select
          drivers.surname,
          count(*) wins
        from drivers
          join results on results.driverid = drivers.driverid and results.position = 1
          join races using(raceid)
        where extract('year' from date_trunc('decade', races.date)) = decades.decade
        group by decades.decade, drivers.driverid
        order by wins desc
        limit 3
    ) as lat on true
  order by decade, wins desc
;
```

and the results:
```
 decade │  surname   │ rank │ wins
════════╪════════════╪══════╪══════
   1950 │ Fangio     │    1 │   24
   1950 │ Ascari     │    2 │   13
   1950 │ Moss       │    3 │   12
   1960 │ Clark      │    1 │   25
   1960 │ Hill       │    2 │   14
   1960 │ Stewart    │    3 │   11
   1970 │ Lauda      │    1 │   17
   1970 │ Stewart    │    2 │   16
   1970 │ Fittipaldi │    3 │   14
   1980 │ Prost      │    1 │   39
   1980 │ Senna      │    2 │   20
   1980 │ Piquet     │    2 │   20
   1990 │ Schumacher │    1 │   35
   1990 │ Hill       │    2 │   22
   1990 │ Senna      │    3 │   21
   2000 │ Schumacher │    1 │   56
   2000 │ Alonso     │    2 │   21
   2000 │ Räikkönen  │    3 │   18
   2010 │ Hamilton   │    1 │   51
   2010 │ Vettel     │    2 │   42
   2010 │ Rosberg    │    3 │   23
```

Similar situations that warrant the use of lateral joins: __Create a report informing the last two sells of each game__.
