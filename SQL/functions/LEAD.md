# LEAD

Oracle documentation: https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions086.htm#SQLRF00656

LEAD is a way to access another row of the table you are querying without actually performing a self join.

Imagine the use case you want to a report on employees, and for each employee you list, you also want to know the most previous hire.

Without the use of `LEAD` you could do something like:

```sql
with emp_and_prev as (
  select
      emp1.first_name
    , emp1.last_name
    , emp1.hire_date
    , emp2.first_name prev_first_name
    , emp2.last_name prev_last_name
    , emp2.hire_date prev_hire_date
    , row_number() over (partition by emp1.employee_id order by emp2.hire_date desc) prev_hire_ranked
  from
      employees emp1
    , employees emp2
  where
      emp2.hire_date < emp1.hire_date
)

select
    first_name
  , last_name
  , hire_date
  , prev_first_name
  , prev_last_name
  , prev_hire_date
from emp_and_prev
where prev_hire_ranked = 1
```

Now we can avoid that self join like so:

```sql
select
    first_name
  , last_name
  , hire_date
  , lead(first_name) over (order by hire_Date desc) prev_first_name
  , lead(last_name) over (order by hire_Date desc) prev_last_name
  , lead(hire_date) over (order by hire_Date desc) prev_hire_date
from employees
```

Producing a lot cleaner query.

In the above example, I just passed in one parameter to the lead function, but it accepts:

* Column - the value to be returned
* Offset - how many rows to offset by, given the order by clause. The defaults to 1, if left off
* Default - what to return in the case of NULL

You can also respect (default) or ignore NULLs in the test with the syntax: `lead(column IGNORE NULLS)` or `lead(column) IGNORE NULLS`.
