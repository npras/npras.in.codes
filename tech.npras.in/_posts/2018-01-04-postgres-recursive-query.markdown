---
layout: post
title: "PostgreSQL notes: Recursive queries vs standard ORM"
excerpt: "See the power of declarative SQL vs the limitations of imperative code"
---


Consider this usecase. You have an `employees` table. Each employee has one manager who's also an employee. Now, given an employee id, it would be useful to list out all of his subordinates or ancestors. Not just the immediate ones. We need all the names reaching all the way up or all the way down.

Let's do it first using ruby's ActiveRecord ORM and then using raw SQL query using postgresql's recursive query feature.

First we need a table with data. Run the following DDL queries in `psql`:

```sql
CREATE TABLE employees (
   employee_id serial PRIMARY KEY,
   full_name VARCHAR NOT NULL,
   manager_id INT
);

INSERT INTO employees (
  employee_id,
  full_name,
  manager_id
)
VALUES
(1, 'Founder', NULL),
(2, 'Exec1', 1),
(3, 'Exec2', 1),
(4, 'Exec3', 1),
(5, 'Exec4', 1),
(6, 'Manager1', 2),
(7, 'Manager2', 2),
(8, 'Manager3', 2),
(9, 'Manager4', 2),
(10, 'Manager5', 3),
(11, 'Manager6', 3),
(12, 'Manager7', 3),
(13, 'Manager8', 3),
(14, 'Manager9', 4),
(15, 'Manager10', 4),
(16, 'Senior Engineer1', 7),
(17, 'Senior Engineer2', 7),
(18, 'Senior Engineer3', 8),
(19, 'Senior Engineer4', 8),
(20, 'Senior Engineer5', 8),
(21, 'Engineer1', 16)
;
```

As you can see from the data, we have a hierarchical structure where 1 employee has 1 manager, but by inference, has many "ancestor" managers. Similarly, a manager employee can have 1 or more immediate subordinates under him, and by inference, has many subordinates through his direct subordinates.


### Expected output
Given an employee `Exec1`, I want to list all of his subordinates, even all the ones he's not directly managing.

Here's the output:
```
 employee_id │ manager_id │    full_name
═════════════╪════════════╪══════════════════
           2 │          1 │ Exec1
           6 │          2 │ Manager1
           7 │          2 │ Manager2
           8 │          2 │ Manager3
           9 │          2 │ Manager4
          16 │          7 │ Senior Engineer1
          17 │          7 │ Senior Engineer2
          18 │          8 │ Senior Engineer3
          19 │          8 │ Senior Engineer4
          20 │          8 │ Senior Engineer5
          21 │         16 │ Engineer1
```

Given an employee `Engineer1`, I want to list all of his ancestors (ok, managers actually!), even all the ones he's not directly reporting to.

Here's the output:
```
 employee_id │ manager_id │    full_name
═════════════╪════════════╪══════════════════
          21 │         16 │ Engineer1
          16 │          7 │ Senior Engineer1
           7 │          2 │ Manager2
           2 │          1 │ Exec1
           1 │          ¤ │ Founder
```

### The ORM way
The official [rails guides lists this very same example and usecase](http://edgeguides.rubyonrails.org/association_basics.html#self-joins) and suggests self-join as the way to get a particular employee's subordinates. But this only lists the immediate subordinates.

To get all the subordinates or ancestors of a given employee, we'll have to resort to writing code that recursively aggregates the employees in an array and returns.

Here's a standalone ruby file that uses activerecord and pg gems to connect to the table that we created above and calculate the answers for our usecases.

```rb
require 'pg'
require 'active_record'

ActiveRecord::Base.logger = Logger.new(STDOUT)

ActiveRecord::Base.establish_connection(
  adapter: 'postgresql',
  host: 'localhost',
  database: 'database',
  username: 'user',
  password: 'password',
)


class Employee < ActiveRecord::Base
  self.primary_key = :employee_id

  has_many :ancestors, class_name: "Employee", primary_key: "manager_id"
  has_many :subordinates, class_name: "Employee", foreign_key: "manager_id"
 
  belongs_to :manager, class_name: "Employee"

  def all_subordinates
    subordinates.map do |sub|
      [sub] + sub.all_subordinates
    end.flatten
  end

  def self_and_subordinates
    [self] + all_subordinates
  end

  def all_ancestors
    ancestors.map do |anc|
      anc.all_ancestors + [anc]
    end.flatten
  end

  def self_and_ancestors
    [self] + all_ancestors
  end
end


exec1 = Employee.find(2)
engineer1 = Employee.find(21)

# Subordinates of exec1
exec1.self_and_subordinates.sort_by {|s| s.employee_id}.each do |e|
  puts "#{e.employee_id} \t #{e.manager_id} \t\t #{e.full_name}"
end

puts
puts

# Ancestors of engineer1
engineer1.self_and_ancestors.sort {|a, b| b.employee_id <=> a.employee_id}.each do |e|
  puts "#{e.employee_id} \t #{e.manager_id} \t\t #{e.full_name}"
end
```

We get the expected answers, but at the cost of several queries fired from from the ruby process to the database server. It actually runs 1 query for fetching each ancestor. So if your org tree is deep, this would perform that many queries. Here's the output from the log when the code is getting all subordinates of `Exec1`:

```log
D, [2018-01-04T19:28:55.876221 #47452] DEBUG -- :   Employee Load (0.3ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 2]]
D, [2018-01-04T19:28:55.877315 #47452] DEBUG -- :   Employee Load (0.3ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 6]]
D, [2018-01-04T19:28:55.878340 #47452] DEBUG -- :   Employee Load (0.3ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 7]]
D, [2018-01-04T19:28:55.879332 #47452] DEBUG -- :   Employee Load (0.3ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 16]]
D, [2018-01-04T19:28:55.880300 #47452] DEBUG -- :   Employee Load (0.3ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 21]]
D, [2018-01-04T19:28:55.881195 #47452] DEBUG -- :   Employee Load (0.3ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 17]]
D, [2018-01-04T19:28:55.882148 #47452] DEBUG -- :   Employee Load (0.3ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 8]]
D, [2018-01-04T19:28:55.883202 #47452] DEBUG -- :   Employee Load (0.2ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 18]]
D, [2018-01-04T19:28:55.884117 #47452] DEBUG -- :   Employee Load (0.2ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 19]]
D, [2018-01-04T19:28:55.885398 #47452] DEBUG -- :   Employee Load (0.4ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 20]]
D, [2018-01-04T19:28:55.886675 #47452] DEBUG -- :   Employee Load (0.4ms)  SELECT "employees".* FROM "employees" WHERE "employees"."manager_id" = $1  [["manager_id", 9]]
```

### The SQL way

Now contrast this with the postgressql's recursive query. All we need is 1 query to get either the subordinates or ancestors.

The query to get subordinates:
```sql
WITH RECURSIVE subordinates AS (
   SELECT
       employee_id,
       manager_id,
       full_name
     FROM employees
     WHERE employee_id = 2
   UNION
   SELECT
       e.employee_id,
       e.manager_id,
       e.full_name
     FROM employees e
       INNER JOIN subordinates s ON s.employee_id = e.manager_id
)
SELECT
    *
  FROM subordinates
;
```

The query to get ancestors:
```sql
WITH RECURSIVE ancestors AS (
   SELECT
       employee_id,
       manager_id,
       full_name
     FROM employees
     WHERE employee_id = 21
   UNION
   SELECT
       e.employee_id,
       e.manager_id,
       e.full_name
     FROM employees e
       INNER JOIN ancestors a ON a.manager_id = e.employee_id
)
SELECT
    *
  FROM ancestors
;
```

To actually know how the recursive query works check this [nice article](http://www.postgresqltutorial.com/postgresql-recursive-query/).


### Conclusion

Of course, now that you know this capability exists, you could tell activerecord to run this query using `find_by_sql`.

Also, note that while it is technically a single query, internally the planner will run loops of queries recursively (like how we did) to get the answers. But it would still be blazing fast compared to the 'multiple queries' version as there's no network delay involved here. You need to check the `explain plan` of the query just to be sure.

Raw SQL FTW.

<div style="width:25%;height:25%">
  <div style="width:100%;height:0;padding-bottom:135%;position:relative;"><iframe src="https://giphy.com/embed/nXxOjZrbnbRxS" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div><p><a href="https://giphy.com/gifs/win-nXxOjZrbnbRxS">via GIPHY</a></p>
</div>
