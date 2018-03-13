---
layout: post
title: "PostgreSQL notes: Window functions"
excerpt: "Process several values of the result set at a time"
---

I'm currently reading the excellent [__Mastering PostgreSQL__](https://masteringpostgresql.com/) by Dimitri Fontaine. Came across this nice ability to perform `lateral join`. Most of the examples seen below are directly from the book. By re-constructing them by hand, we'll get a better understanding.


### The theory

The whole idea behind window functions is to allow you to process several values of the result set at a time: you see through the window some peer rows and you are able to compute a single output value from them, much like when using an aggregate function.

Remember that the window functions only happens after the where clause, so you only get to see rows from the available result set of the query.

Any and all aggregate function you already know can be used against a window frame rather than a grouping clause, so you can already start to use sum, min, max, count, avg, and the other that you’re already used to using.

Use window functions whenever you want to compute values for each row of the result set and those computations depend on other rows within the same result set. A classic example is a marketing analysis of weekly results: you typically output both each day’s gross sales and the variation with the same day in comparison to the previous week.

row_number() vs rank() vs dense_rank() window functions: refer this: https://dzone.com/articles/difference-between-rownumber


### The basics

empty `over()` means a single partition of the whole window of result:
```
chinook=# select x, array_agg(x) over () from generate_series(1, 3) t(x);
 x | array_agg
---+-----------
 1 | {1,2,3}
 2 | {1,2,3}
 3 | {1,2,3}
```

Rolling aggregate of a column's value:
```
chinook=# select x, array_agg(x) over (order by x) from generate_series(1, 3) t(x);
 x | array_agg
---+-----------
 1 | {1}
 2 | {1,2}
 3 | {1,2,3}
```

Sum of all values in a column mentioned in all rows throughout (unlike in a single row returned by the normal `sum(x)` aggregate function):
```
chinook=# select x, sum(x) over () from generate_series(1, 3) t(x);
 x | sum
---+-----
 1 |   6
 2 |   6
 3 |   6
```

Rolling sum. Sum up until that particular row:
```
chinook=# select x, sum(x) over (order by x) from generate_series(1, 3) t(x);
 x | sum
---+-----
 1 |   1
 2 |   3
 3 |   6
```

percentage of a row value in the total values:
```
select x,
       array_agg(x) over () as frame,
       sum(x) over () as sum,
       x::float * 100/(sum(x) over ()) as part
  from generate_series(1, 3) as t(x);

x |  frame  | sum |       part
---+---------+-----+------------------
 1 | {1,2,3} |   6 | 16.6666666666667
 2 | {1,2,3} |   6 | 33.3333333333333
 3 | {1,2,3} |   6 |               50
```

__Pause and reflect on the above__. It's a breakthrough. you could compute both the total sum of a column and the ratio of the current value compared to the this total within a single SQL query.


### The examples

__Given a playlistId, list all tracks in it along with the "nextSong" and "previousSong" cols.__

Expected output:
```
       playlist_name        |      track_name      |       prevsong       |       nextsong
----------------------------+----------------------+----------------------+----------------------
 Classical 101 - The Basics | Intoitus: Adorate De |                      | Miserere mei, Deus
 Classical 101 - The Basics | Miserere mei, Deus   | Intoitus: Adorate De | Canon and Gigue in D
 Classical 101 - The Basics | Canon and Gigue in D | Miserere mei, Deus   | Concerto No. 1 in E
 Classical 101 - The Basics | Concerto No. 1 in E  | Canon and Gigue in D | Concerto for 2 Violi
 Classical 101 - The Basics | Concerto for 2 Violi | Concerto No. 1 in E  | Aria Mit 30 Ver�nder
 Classical 101 - The Basics | Aria Mit 30 Ver�nder | Concerto for 2 Violi | Suite for Solo Cello
 Classical 101 - The Basics | Suite for Solo Cello | Aria Mit 30 Ver�nder | The Messiah: Behold,
 Classical 101 - The Basics | The Messiah: Behold, | Suite for Solo Cello | Solomon HWV 67: The
 Classical 101 - The Basics | Solomon HWV 67: The  | The Messiah: Behold, | "Eine Kleine Nachtmu
 Classical 101 - The Basics | "Eine Kleine Nachtmu | Solomon HWV 67: The  | Concerto for Clarine
 Classical 101 - The Basics | Concerto for Clarine | "Eine Kleine Nachtmu | Symphony No. 104 in
 Classical 101 - The Basics | Symphony No. 104 in  | Concerto for Clarine | Symphony No.5 in C M
 Classical 101 - The Basics | Symphony No.5 in C M | Symphony No. 104 in  | Ave Maria
 Classical 101 - The Basics | Ave Maria            | Symphony No.5 in C M | Nabucco: Chorus, "Va
 Classical 101 - The Basics | Nabucco: Chorus, "Va | Ave Maria            | Die Walk�re: The Rid
 Classical 101 - The Basics | Die Walk�re: The Rid | Nabucco: Chorus, "Va | Requiem, Op.48: 4. P
 Classical 101 - The Basics | Requiem, Op.48: 4. P | Die Walk�re: The Rid | The Nutcracker, Op.
 Classical 101 - The Basics | The Nutcracker, Op.  | Requiem, Op.48: 4. P | Nimrod (Adagio) from
 Classical 101 - The Basics | Nimrod (Adagio) from | The Nutcracker, Op.  | Madama Butterfly: Un
 Classical 101 - The Basics | Madama Butterfly: Un | Nimrod (Adagio) from | Jupiter, the Bringer
 Classical 101 - The Basics | Jupiter, the Bringer | Madama Butterfly: Un | Turandot, Act III, N
 Classical 101 - The Basics | Turandot, Act III, N | Jupiter, the Bringer | Adagio for Strings f
 Classical 101 - The Basics | Adagio for Strings f | Turandot, Act III, N | Carmina Burana: O Fo
 Classical 101 - The Basics | Carmina Burana: O Fo | Adagio for Strings f | Fanfare for the Comm
 Classical 101 - The Basics | Fanfare for the Comm | Carmina Burana: O Fo |
```

And the sql is:
```sql
select
    playlist."Name" playlist_name,
    left(track."Name", 20) track_name,
    left(lag(track."Name", 1) over (), 20) prevSong,
    left(lead(track."Name", 1) over (), 20) nextSong
  from "Playlist" playlist
    join "PlaylistTrack" using("PlaylistId")
    join "Track" track using("TrackId")
  where playlist."PlaylistId" = 13
;
```

---

__Given a playlist id, list all of its tracks with individual duration as well as rolling sum of duration.__

The expected output:
```
       playlist_name        |      track_name      |   duration   |   progress
----------------------------+----------------------+--------------+--------------
 Classical 101 - The Basics | Aria Mit 30 Ver�nder | 00:02:00.463 | 00:02:00.463
 Classical 101 - The Basics | Suite for Solo Cello | 00:02:23.288 | 00:04:23.751
 Classical 101 - The Basics | Carmina Burana: O Fo | 00:02:36.71  | 00:07:00.461
 Classical 101 - The Basics | Turandot, Act III, N | 00:02:56.911 | 00:09:57.372
 Classical 101 - The Basics | Die Walk�re: The Rid | 00:03:09.008 | 00:13:06.38
 Classical 101 - The Basics | Concerto for 2 Violi | 00:03:13.722 | 00:16:20.102
 Classical 101 - The Basics | Solomon HWV 67: The  | 00:03:17.135 | 00:19:37.237
 Classical 101 - The Basics | Fanfare for the Comm | 00:03:18.064 | 00:22:55.301
 Classical 101 - The Basics | Concerto No. 1 in E  | 00:03:19.086 | 00:26:14.387
 Classical 101 - The Basics | Intoitus: Adorate De | 00:04:05.317 | 00:30:19.704
 Classical 101 - The Basics | Nimrod (Adagio) from | 00:04:10.031 | 00:34:29.735
 Classical 101 - The Basics | Requiem, Op.48: 4. P | 00:04:18.924 | 00:38:48.659
 Classical 101 - The Basics | Canon and Gigue in D | 00:04:31.788 | 00:43:20.447
 Classical 101 - The Basics | Nabucco: Chorus, "Va | 00:04:34.504 | 00:47:54.951
 Classical 101 - The Basics | Madama Butterfly: Un | 00:04:37.639 | 00:52:32.59
 Classical 101 - The Basics | The Nutcracker, Op.  | 00:05:04.226 | 00:57:36.816
 Classical 101 - The Basics | Symphony No. 104 in  | 00:05:06.687 | 01:02:43.503
 Classical 101 - The Basics | Ave Maria            | 00:05:38.243 | 01:08:21.746
 Classical 101 - The Basics | "Eine Kleine Nachtmu | 00:05:48.971 | 01:14:10.717
 Classical 101 - The Basics | Symphony No.5 in C M | 00:06:32.462 | 01:20:43.179
 Classical 101 - The Basics | Concerto for Clarine | 00:06:34.482 | 01:27:17.661
 Classical 101 - The Basics | Miserere mei, Deus   | 00:08:21.503 | 01:35:39.164
 Classical 101 - The Basics | Jupiter, the Bringer | 00:08:42.099 | 01:44:21.263
 Classical 101 - The Basics | The Messiah: Behold, | 00:09:42.029 | 01:54:03.292
 Classical 101 - The Basics | Adagio for Strings f | 00:09:56.519 | 02:03:59.811
```

And the sql is:
```sql
select
    playlist."Name" playlist_name,
    left(track."Name", 20) track_name,
    track."Milliseconds" * interval '1 ms' duration,
    (sum(track."Milliseconds") over (order by track."Milliseconds")) * interval '1 ms' progress
  from "Playlist" playlist
    join "PlaylistTrack" using("PlaylistId")
    join "Track" track using("TrackId")
    where playlist."PlaylistId" = 15
;
```

---

Consider the `f1db`. It has the `results` table with all race results in history. Suppose we want to list all participants in the order of their final `position` - winner at top. And we also want a column that shows the position of the driver among his own teammates. Eg: If in a race, there were 5 Ferrari cars, then this field would list values like `1/5` for a driver, meaning, this driver came first of all of the ferrari drivers.

This query was written using row_number() and count(\*) as window functions.

Expected output:
```
    surname    │ constructorname │ position │ pos same constr
═══════════════╪═════════════════╪══════════╪═════════════════
 Hamilton      │ Mercedes        │        1 │ 1 / 2
 Räikkönen     │ Lotus F1        │        2 │ 1 / 2
 Vettel        │ Red Bull        │        3 │ 1 / 2
 Webber        │ Red Bull        │        4 │ 2 / 2
 Alonso        │ Ferrari         │        5 │ 1 / 2
 Grosjean      │ Lotus F1        │        6 │ 2 / 2
 Button        │ McLaren         │        7 │ 1 / 2
 Massa         │ Ferrari         │        8 │ 2 / 2
 Pérez         │ McLaren         │        9 │ 2 / 2
 Maldonado     │ Williams        │       10 │ 1 / 2
 Hülkenberg    │ Sauber          │       11 │ 1 / 2
 Vergne        │ Toro Rosso      │       12 │ 1 / 2
 Ricciardo     │ Toro Rosso      │       13 │ 2 / 2
 van der Garde │ Caterham        │       14 │ 1 / 2
 Pic           │ Caterham        │       15 │ 2 / 2
 Bianchi       │ Marussia        │       16 │ 1 / 2
 Chilton       │ Marussia        │       17 │ 2 / 2
 di Resta      │ Force India     │       18 │ 1 / 2
 Rosberg       │ Mercedes        │       19 │ 2 / 2
 Bottas        │ Williams        │        ¤ │ 2 / 2
 Sutil         │ Force India     │        ¤ │ 2 / 2
 Gutiérrez     │ Sauber          │        ¤ │ 2 / 2
```

And the sql is:
```sql
select
    surname,
    constructors.name constructorName,
    position,
    format('%s / %s',
          row_number() over (partition by constructorid order by position nulls last),
          count(*) over (partition by constructorid)
          ) "pos same constr"
  from results
    join drivers using(driverid)
    join constructors using(constructorid)
  where raceid = 890
  order by position
;
```

---

__Suppose, you has a list of f1 race winners displayed with based on their final winning position - winner on top. But you also want to show a column with the rank numbers based on "fastestlapspeed". And, you also want to group the winners into 5 equal groups (for whatever reason).__

The expected output:
```
    surname    | position | fastest | group |   previous    |     next
---------------+----------+---------+-------+---------------+---------------
 Hamilton      |        1 |      20 |     1 |               | Räikkönen
 Räikkönen     |        2 |      17 |     1 | Hamilton      | Vettel
 Vettel        |        3 |      21 |     1 | Räikkönen     | Webber
 Webber        |        4 |      22 |     1 | Vettel        | Alonso
 Alonso        |        5 |      15 |     1 | Webber        | Grosjean
 Grosjean      |        6 |      16 |     2 | Alonso        | Button
 Button        |        7 |      12 |     2 | Grosjean      | Massa
 Massa         |        8 |      18 |     2 | Button        | Pérez
 Pérez         |        9 |      13 |     2 | Massa         | Maldonado
 Maldonado     |       10 |      14 |     2 | Pérez         | Hülkenberg
 Hülkenberg    |       11 |       9 |     3 | Maldonado     | Vergne
 Vergne        |       12 |      11 |     3 | Hülkenberg    | Ricciardo
 Ricciardo     |       13 |       8 |     3 | Vergne        | van der Garde
 van der Garde |       14 |       6 |     3 | Ricciardo     | Pic
 Pic           |       15 |       5 |     4 | van der Garde | Bianchi
 Bianchi       |       16 |       3 |     4 | Pic           | Chilton
 Chilton       |       17 |       4 |     4 | Bianchi       | di Resta
 di Resta      |       18 |      10 |     4 | Chilton       | Rosberg
 Rosberg       |       19 |      19 |     5 | di Resta      | Bottas
 Sutil         |          |       2 |     5 | Gutiérrez     |
 Gutiérrez     |          |       1 |     5 | Bottas        | Sutil
 Bottas        |          |       7 |     5 | Rosberg       | Gutiérrez
 ```

 And the sql is:
```sql
select surname,
         position,
         row_number() over(order by fastestlapspeed::numeric) as "fastest",
         ntile(5) over w as "group",
         lag(surname, 1) over w as "previous",
         lead(surname, 1) over w as "next"
    from      results
         join drivers using(driverid)
   where raceid = 890
  window w as (order by position)
order by position;
```

---

 __List total number of races for each decade, along with the difference in value with the preceding year.__

 The expected output:
```
 decade │ nbraces │ evolution
════════╪═════════╪═══════════
   1950 │      84 │         ¤
   1960 │     100 │        16
   1970 │     144 │        44
   1980 │     156 │        12
   1990 │     162 │         6
   2000 │     174 │        12
   2010 │     177 │         3
```
 
And the sql is:
```sql
 with rbd as
(
select
    extract('year' from date_trunc('decade', date)) as decade,
    count(*) nbraces
  from races
  group by decade
  order by decade
)

select
    rbd.decade,
    rbd.nbraces,
    (rbd.nbraces - lag(nbraces, 1) over (order by decade)) evolution
  from rbd
;
```
