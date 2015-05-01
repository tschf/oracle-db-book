# LAST_VALUE

LAST_VALUE is an analytic function. Given and order by clause, you can find the last value in a set of rows.

```sql
select
    department_id
  , employee_id
  , hire_date
  , last_value(hire_date) over (partition by department_id order by hire_date rows between unbounded preceding and unbounded following) last_hire_date
from employees
```

```
DEPARTMENT_ID EMPLOYEE_ID HIRE_DATE LAST_HIRE_DATE
------------- ----------- --------- --------------
           10         200 17/SEP/03 17/SEP/03
           20         201 17/FEB/04 17/AUG/05  
           20         202 17/AUG/05 17/AUG/05
```

You may also be interested in the [FIRST_VALUE](FIRST_VALUE.md) function which gets the first value in a set of rows.
