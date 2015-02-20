# MIN

min is an aggregate function, to get the smallest value amongst a set of rows.

```sql
select min(hire_date) min_hire
from employees
```
Output:
```
MIN_HIRE
--------------
13/JAN/01
```

```sql
select department_id, min(hire_date) min_hire
from employees
group by department_id
```

Output:
```
DEPARTMENT_ID MIN_HIRE
------------- ---------
          100 16/AUG/02
           30 07/DEC/02
              24/MAY/07
           90 13/JAN/01
           20 17/FEB/04
           70 07/JUN/02
          110 07/JUN/02
           50 01/MAY/03
           80 30/JAN/04
           40 07/JUN/02
           60 25/JUN/05
           10 17/SEP/03
```

It can also be used as an analytic function:

```sql
select
    department_id
  , employee_id
  , hire_date
  , min(hire_date) over(partition by department_id order by hire_date) first_hire
from employees
```

Output:
```
DEPARTMENT_ID EMPLOYEE_ID HIRE_DATE FIRST_HIRE
------------- ----------- --------- ----------
           10         200 17/SEP/03 17/SEP/03  
           20         201 17/FEB/04 17/FEB/04  
           20         202 17/AUG/05 17/FEB/04  
           30         114 07/DEC/02 07/DEC/02  
           30         115 18/MAY/03 07/DEC/02  
           30         117 24/JUL/05 07/DEC/02

107 rows selected
```

You may think of the least function, however these are different in min is an aggregate or analytic function operating on a set of rows - least operates on a parameterised list of values. [Read more here](LEAST.md).
