# ITERATION_NUMBER

`iteration_number` is used when you make use of SQL modelling. In particular, when you specify to iterate, this function returns which iteration number you are on, starting from zero.

```sql
select
    employee_id,
    first_name
from employees
where employee_id >= 200
model
    return updated rows
    dimension by (employee_id)
    measures (first_name)
    rules iterate(3)(

        first_name[any] = first_name[cv()]||'-'||iteration_number
    )
order by employee_id
```

Output:
```
EMPLOYEE_ID FIRST_NAME
----------- --------------------
        200 Jennifer-0-1-2
        201 Michael-0-1-2
        202 Pat-0-1-2
        203 Susan-0-1-2
        204 Hermann-0-1-2
        205 Shelley-0-1-2
        206 William-0-1-2

 7 rows selected
```
