# COUNT

count is an aggregate function, to get the total number of rows in a given set of rows.

```sql
select count(*) employee_count
from employees
```
Output:
```
EMPLOYEE_COUNT
--------------
           107
```

```sql
select department_id, count(*) department_count
from employees
group by department_id
```

Output:
```
DEPARTMENT_ID DEPARTMENT_COUNT
------------- ----------------
          100                6
           30                6
                             1
           90                3
           20                2
           70                1
          110                2
           50               45
           80               34
           40                1
           60                5
           10                1
```

It can also be used as an analytic function:

```sql
select
    department_id
  , count(*) over (partition by department_id order by department_id) emp_count
from employees
```

Output:
```
DEPARTMENT_ID  EMP_COUNT
------------- ----------
           10          1
           20          2
           20          2
           30          6
           30          6
           30          6
           30          6
           30          6
           30          6
           40          1
           50         45

107 rows selected
```
