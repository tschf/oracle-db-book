# NTILE

Oracle documentation: https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions115.htm#SQLRF00680

NTILE is an analytic function. It accepts one parameter - n - which determines which quartile the row falls in - up to a maximum of n, given by an order by clause in the analytic expression.

If the number of rows is less than n, ntile will return the row number, as it can't satisfy all quartiles.

E.g. you may wish to report on the 20% most recent hires. For this particular example, since some departments have less than 5 (100/20%) employees, it is important to order in descending order by the hire date, so we can use `1` in the filter condition.

```sql
with emp_hire_quartiles as (
select
    employee_id
  , department_id  
  , first_name
  , last_name
  , hire_date
  , ntile(100/20) over (partition by department_id order by hire_date desc) hire_quartile
from employees)
select
    employee_id
  , department_id  
  , first_name
  , last_name
  , hire_date
from
    emp_hire_quartiles
where
   hire_quartile = 1
```
```
EMPLOYEE_ID DEPARTMENT_ID FIRST_NAME           LAST_NAME                 HIRE_DATE
----------- ------------- -------------------- ------------------------- ---------
        200            10 Jennifer             Whalen                    17/SEP/03
        202            20 Pat                  Fay                       17/AUG/05
        119            30 Karen                Colmenares                10/AUG/07
        118            30 Guy                  Himuro                    15/NOV/06
        203            40 Susan                Mavris                    07/JUN/02
        128            50 Steven               Markle                    08/MAR/08
        136            50 Hazel                Philtanker                06/FEB/08
        183            50 Girard               Geoni                     03/FEB/08
```
