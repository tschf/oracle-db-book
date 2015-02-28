# MAX

`max` is an aggregate function, to get the smallest value amongst a set of rows.

```sql
select min(hire_date) max_hire
from employees
```
Output:
```
MAX_HIRE
---------
13/JAN/01
```

```sql
select department_id, max(hire_date) max_hire
from employees
group by department_id
```

Output:
```
DEPARTMENT_ID MAX_HIRE
------------- ---------
          100 07/DEC/07
           30 10/AUG/07
              24/MAY/07
           90 21/SEP/05
           20 17/AUG/05
           70 07/JUN/02
          110 07/JUN/02
           50 08/MAR/08
           80 21/APR/08
           40 07/JUN/02
           60 21/MAY/07
           10 17/SEP/03
```

It can also be used as an analytic function:

```sql
select
    department_id
  , employee_id
  , hire_date
  , max(hire_date) over(partition by department_id) last_hire
from employees
```

Output:
```
DEPARTMENT_ID EMPLOYEE_ID HIRE_DATE LAST_HIRE
------------- ----------- --------- ---------
           10         200 17/SEP/03 17/SEP/03
           20         201 17/FEB/04 17/AUG/05
           20         202 17/AUG/05 17/AUG/05
           30         114 07/DEC/02 10/AUG/07
           30         115 18/MAY/03 10/AUG/07
           30         116 24/DEC/05 10/AUG/07

107 rows selected
```

You may think of the `greatest` function, however these are different in `max` is an aggregate or analytic function operating on a set of rows - `greatest` operates on a parameterised list of values. [Read more here](GREATEST.md).
