---
layout: post
title: "PostgreSQL notes: A case for writing business logic in sql rather than code"
excerpt: "All future programming will be declarative. SQL was defined as declarative 20 years ago"
---

I'm currently reading the excellent [__Mastering PostgreSQL__](https://masteringpostgresql.com/) by Dimitri Fontaine. The book is specifically targetted at application developers like me. His main argument is that, in most of the applications, the business logic involves the data, and so a lot of those logic can be written *in sql* in the database. And he specifically sells PostgreSQL over others like MySQL.

And I'm sold. Actually I was sold a long time back when [Derek Sivers](https://sivers.org/pg) wrote about the exact same topic.

But Dimitri gives a lot of details and examples that are convincing and models real world usecases. And he doesn't explain things like triggers and stored procedures. He says, just *a query* is enough to do a lot of business logic heavy-lifting.

In the sample chapter of the book, he explains a scenario where the business wants to see the "week over week difference as percentage" for a `factbook` table. That is, suppose today is Sunday and today's dollars value is 64.52. And I want to compare it with last Sunday's value, which was 64.61. I want the comparison as a difference percentage, that is: 
 `-0.14%`

 The problem here: the database table only has the dollars value. We have to calculate the WoW%, either in code or in sql.

 Here, I've implemented a solution in ruby, using `Hash Join Nested Loop`.

 Compare it with how Dimitri does it with just sql, using `Merge Left Join over two ordered relations`.

 At first, the ruby solution might be easy to reason about. It's imperative and it's like we are hand-holding the computer as to what steps to take during the entire journey.

 But once you get a hold of the different concepts used in the sql query, _and_ know the exact sql tools to reach out for solving a specific problem, then you wouldn't go back to code to write this logic!

 Pound for pound, the sql query remains the most dense and concise solution.
 
 The several PostgreSQL features used in this query are:

 * Computed Table Expressions or CTEs: Using this you could create a __virtual relation__ based on a SELECT, and then use this relation in conjunction with other existing relations.
 * Window functions: The `lag... over...` window function lets us compare the current row with aa specific row before it based on some partition condition. In this case we are comparing the current row with the previous row that has the same weekday. (ie, comparing a Wednesday row with previous Wednesday row).
 * We are joining an actual table `factbook` with a virtual table generated with `generate_series`.
 * We are using the shorthand join syntax `using`.
 * `COALESCE` function returns the first non-null argument passed to it. This allows us to pass a default value in case a NULL is present.
 * to_char(date, 'Dy') is a string formatting function that, when passed a date, returns a shorthand day notation. Eg: Mon, Tue.
 * CASE..WHEN..THEN..END

__My ruby solution__
```rb
require 'sequel'
require 'byebug'

def connect
  Sequel.connect(adapter: 'postgres', host: 'localhost', database: 'chinook', user: 'user', password: 'password')
end


def fetch_month_data(year:, month:)
  start_date = "%d-%02d-01" % [year, month]
  sql = <<~SQL
  select date, shares, trades, dollars
  from factbook
  where date >= (date '#{start_date}' - interval '1 week')
    and date < date '#{start_date}' + interval '1 month'
  order by date;
  SQL
  dataset = DB[:factbook].with_sql(sql).all
  dataset.each_with_object({}) do |row_hash, obj|
    obj[row_hash[:date].to_s] = row_hash.values_at(:shares, :trades, :dollars)
  end
end


def add_last_week_dollars(data)
  data.keys.sort.each do |day|
    the_week_before = (Date.parse(day) - 7).to_s
    data[day] << (data[the_week_before] ? data[the_week_before][2] : nil)
  end
  data
end


def list_book_for_month(year:, month:)
  data = fetch_month_data(year: year, month: month)
  data = add_last_week_dollars(data)

  printf("%12s | %12s | %12s | %12s | %12s", *%w(day shares trades dollars wow%))
  puts
  printf("%12s-+-%12s-+-%12s-+-%12s-+-%12s", *Array.new(5, '-' * 12))
  puts

  start_date = Date.new(year.to_i, month.to_i, 1)# - 7
  (start_date..Date.new(year.to_i, month.to_i, -1)).each do |day|
    if data[day.to_s]
      shares, trades, dollars, last_week_dollars = data[day.to_s]
    else
      shares, trades, dollars, last_week_dollars = [0, 0, 0, nil]
    end
    wow = (!dollars.zero? && last_week_dollars) ? ((dollars.to_f - last_week_dollars) * 100 / dollars).round(2) : nil
    printf("%12s | %12s | %12s | %12s | %12s", *[day, shares, trades, dollars, wow])
    puts
  end
end


if __FILE__ == $PROGRAM_NAME
  DB = connect
  year = ARGV[0]
  month = ARGV[1]
  res = list_book_for_month(year: year, month: month)
end
```


__Dimitri's sql solution__
```sql
with computed_data as
(
  select cast(date as date) as date,
         to_char(date, 'Dy') as day,
         coalesce(dollars, 0) as dollars,
         lag(dollars, 1)
           over(
             partition by extract('isodow' from date)
               order by date
           )
         as last_week_dollars
    from generate_series(date :'start' - interval '1 week',
                         date :'start' + interval '1 month' - interval '1 day',
                         interval '1 day') as calendar(date)
      left join factbook using(date)
)

select date, day,
        to_char(coalesce(dollars, 0), 'L99G999G999G999') as dollars,
        case when dollars is not null and dollars <> 0
             then round(100.0 * (dollars - last_week_dollars) / dollars, 2)
        end
       as "WoW %"
  from computed_data
  where date >= date :'start'
  order by date;
```


```psql
    date    | day |     dollars      | WoW %
------------+-----+------------------+--------
 2017-02-01 | Wed | $ 44,660,060,305 |  -2.21
 2017-02-02 | Thu | $ 43,276,102,903 |   1.71
 2017-02-03 | Fri | $ 42,801,562,275 |  10.86
 2017-02-04 | Sat | $              0 |
 2017-02-05 | Sun | $              0 |
 2017-02-06 | Mon | $ 37,300,908,120 |  -9.64
 2017-02-07 | Tue | $ 39,754,062,721 | -37.41
 2017-02-08 | Wed | $ 40,491,648,732 | -10.29
 2017-02-09 | Thu | $ 40,169,585,511 |  -7.73
 2017-02-10 | Fri | $ 38,347,515,768 | -11.61
 2017-02-11 | Sat | $              0 |
 2017-02-12 | Sun | $              0 |
 2017-02-13 | Mon | $ 38,745,317,913 |   3.73
 2017-02-14 | Tue | $ 40,737,106,101 |   2.41
 2017-02-15 | Wed | $ 43,802,653,477 |   7.56
 2017-02-16 | Thu | $ 41,956,691,405 |   4.26
 2017-02-17 | Fri | $ 48,862,504,551 |  21.52
 2017-02-18 | Sat | $              0 |
 2017-02-19 | Sun | $              0 |
 2017-02-20 | Mon | $              0 |
 2017-02-21 | Tue | $ 44,416,927,777 |   8.28
 2017-02-22 | Wed | $ 41,137,731,714 |  -6.48
 2017-02-23 | Thu | $ 44,254,446,593 |   5.19
 2017-02-24 | Fri | $ 45,229,398,830 |  -8.03
 2017-02-25 | Sat | $              0 |
 2017-02-26 | Sun | $              0 |
 2017-02-27 | Mon | $ 43,613,734,358 |
 2017-02-28 | Tue | $ 57,874,495,227 |  23.25
(28 rows)
```
