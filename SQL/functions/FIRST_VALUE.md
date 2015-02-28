# FIRST_VALUE

FIRST_VALUE is an analytic function. Given and order by clause, you can find the first value in a set of rows.

You may wish to determine the first hire date for each department, in an employee report

```
select
    department_id
  , employee_id
  , hire_date
  , first_value(hire_date) over (partition by department_id order by hire_date) first_hire_date
from employees
```

You may also be interested in the [LAST_VALUE](LAST_VALUE.md) function which gets the last value in a set of rows.
