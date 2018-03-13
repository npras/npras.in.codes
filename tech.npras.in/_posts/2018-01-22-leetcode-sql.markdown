---
layout: post
title: "Practicing SQL problems from Leetcode"
excerpt: "With the newly learned PostgreSQL in my arsenal"
---


### 185: Department top 3 salaries
[Problem description here](https://leetcode.com/articles/department-top-three-salaries/).

Expected output:

```
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |
| IT         | Joe      | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
+------------+----------+--------+
```

Given these 2 tables: Employee and Department:

```
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
+----+-------+--------+--------------+

+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
```

The solution, using Postgresql's lateral join:

```sql
select
    department.Name department,
    lat.Name employee,
    lat.Salary salary
  from department
    left join lateral
    (
        select
            employee.Name,
            employee.Salary
          from employee
          where employee.DepartmentId = department.Id
          order by employee.Salary desc
          limit 3
    ) lat on true
;
```

---

### 177: n-th highest salary
[Problem description here](https://leetcode.com/problems/nth-highest-salary/).

If there is no nth highest salary, then the query should return null.

Expected output:

```
+------------------------+
| getNthHighestSalary(2) |
+------------------------+
| 200                    |
+------------------------+
```

given a salary table like this:

```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

The postgresql solution using window function `rank` is:

```sql
with ranked_by_salary as (
    select
        id,
        rank() over(order by salary desc) "rank",
        salary
      from employee
)

select
    id,
    rank,
    salary
  from ranked_by_salary
  where rank = 2
;
```

This could also be written using limit offset as:

```sql
select
    id,
    salary
  from employee
  order by salary desc
  limit 1 offset 1
;
```

---

### 262. Trips and Users
[Problem description here](https://leetcode.com/problems/nth-highest-salary/).

Expected output:

```
+------------+-------------------+
|     Day    | Cancellation Rate |
+------------+-------------------+
| 2013-10-01 |       0.33        |
| 2013-10-02 |       0.00        |
| 2013-10-03 |       0.50        |
+------------+-------------------+
```

And the postgresql solution is:

```sql
select
    trips.request_at as Day,
    round((count(*) filter(where trips.status in ('cancelled_by_client', 'cancelled_by_driver'))::float / count(*))::numeric,
         2) as "Cancellation Rate"
  from trips
    join users on trips.client_id = users.users_id and users.role = 'client' and users.banned = 'No'
  where trips.request_at between '2013-10-01' and '2013-10-03'
    -- and trips.Status = 'cancelled_by_client'
  group by Day
;
```

---

### 579. Find Cumulative Salary of an Employee
[Problem description here](https://leetcode.com/articles/find-cumulative-salary-of-an-employee/).

Solution using postgresql's window functions rank and sum:

```sql
select
    employeeid,
    month,
    salary,
    sum(salary) over (partition by employeeid order by salary) as cumsalary
  from
  (
    select
        employeeid,
        month,
        salary,
        rank() over (partition by employeeid order by salary desc) rank,
        sum(salary) over (partition by employeeid order by salary) as cumsalary
      from employeesalaries
  ) as ranked
  where rank > 1
  order by employeeid, month desc
;
```

---

### 615. Average Salary: Departments VS Company
[Problem description here](https://leetcode.com/articles/average-salary-departments-vs-company/).

Solution using postgresql’s CTE (common table expression):
(I wonder if this can be simplified using window functions?)

```sql
with company_avg as (
  select
      to_char(pay_date, 'YYYY-MM') "pay_month",
      avg(amount) "avg_company"
    from l615_salary
      join l615_employee using(employee_id)
    group by pay_month
), dept_avg as (
  select
      to_char(pay_date, 'YYYY-MM') "pay_month",
      department_id,
      avg(amount) "avg_dept"
    from l615_salary
      join l615_employee using(employee_id)
    group by pay_month, department_id
)
select
    pay_month,
    department_id,
    case
    when avg_dept > avg_company then 'higher'
    when avg_dept < avg_company then 'lower'
    when avg_dept = avg_company then 'same'
    end "comparison"
  from company_avg
    join dept_avg using(pay_month)
  order by pay_month desc, department_id
;
```

---

### 601. Human Traffic of Stadium
[Problem description here](https://leetcode.com/problems/human-traffic-of-stadium/description/).

Haven't got this working yet. But I think this problem is similar to what Dimitri had already blogged about [here](http://tapoueh.org/blog/2012/10/reset-counter/).

I've commented in his blog in that page asking him help. But he's yet to reply.

wip solution:

```sql
```
