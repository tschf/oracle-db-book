# Analytic Functions

Analytic functions, otherwise known as windowing functions, allow a function to be applied over a set of rows. Like aggregate functions, they can be applied to all the rows returned from a query - but they can also be applied to a subset of rows.

The syntax of calling an analytic function is: function name OVER (analytic clause)

That analytic clause component is what defines the window used in computing the results of the function.  It consists of the following components:

1. `partition by` clause - given an expression list, separate windows will be created on each distinct group
* `order by` clause - useful for analytic functions such as `row_number` `rank` `dense_rank` - amongst others. The results of the specified functions typically return a number bast on the order by clause. e.g. `row_number` will return a distinct row number, ordering by the `order by` clause.
* window clause.

For each row within a window, you can also further divide the window than what has already been accomplished with the `partition by` clause. It starts with:

* RANGE; or
* ROWS

Then you will typically specify a start point and/or end point with the aid of the `between` keyword.

However, it is worth noting that in order to use range/rows, you must have an `order by` clause and the column expression must be either a numeric or date value. If you specify an `order by` clause, and leave off a range/rows expression, the default behaviour is to included only the range upto the current value (not larger values).

```sql
select
    employee_id,
    last_name,
    hire_date,
    job_id,
    salary,
    sum(salary) over (order by hire_date rows between 1 PRECEDING and 1 following ) range_3
from employees
order by hire_date
```

```
EMPLOYEE_ID LAST_NAME                 HIRE_DATE JOB_ID         SALARY    RANGE_3
----------- ------------------------- --------- ---------- ---------- ----------
        102 De Haan                   13/JAN/01 AD_VP           17000      25300
        206 Gietz                     07/JUN/02 AC_ACCOUNT       8300      35300
        204 Baer                      07/JUN/02 PR_REP          10000      24800
        203 Mavris                    07/JUN/02 HR_REP           6500      28508
        205 Higgins                   07/JUN/02 AC_MGR          12008      27508
        109 Faviet                    16/AUG/02 FI_ACCOUNT       9000      33016
        108 Greenberg                 17/AUG/02 FI_MGR          12008      32008
        114 Raphaely                  07/DEC/02 PU_MAN          11000      30908
```

## RANGE

`range` uses the value of the current row as the basis for what's included in the function call. Given the statement `sum(salary) over (order by salary range unbounded preceding)` it will find all rows where the salary is less than that of the current row, upto the value of the current row (so can included multiple rows in the to portion).

```sql
SELECT
    manager_id,
    last_name,
    salary,
    SUM(salary) OVER (ORDER BY salary range UNBOUNDED PRECEDING) cume_salary
FROM employees
where manager_id = 100
order by salary
```

Output:

```
MANAGER_ID LAST_NAME                     SALARY CUME_SALARY
---------- ------------------------- ---------- -----------
       100 Mourgos                         5800        5800
       100 Vollman                         6500       12300
       100 Kaufling                        7900       20200
       100 Weiss                           8000       28200
       100 Fripp                           8200       36400
       100 Zlotkey                        10500       46900
       100 Cambrault                      11000       68900
       100 Raphaely                       11000       68900
       100 Errazuriz                      12000       80900
```

If you draw attention to Cambrault and Rephaely, who both have the same salary, you will see their cumulative salary is the same. The reason for this is because they both have the same value as specified in the `order by` clause. We can see this would be different if we add another column expression to the `order by` clause.

```sql
SELECT
    manager_id,
    last_name,
    hire_date,
    salary,
    SUM(salary) OVER (ORDER BY salary, hire_date range UNBOUNDED PRECEDING) cume_salary
FROM employees
where manager_id = 100
order by salary, hire_date
```
Output:
```
MANAGER_ID LAST_NAME                 HIRE_DATE     SALARY CUME_SALARY
---------- ------------------------- --------- ---------- -----------
       100 Mourgos                   16/NOV/07       5800        5800
       100 Vollman                   10/OCT/05       6500       12300
       100 Kaufling                  01/MAY/03       7900       20200
       100 Weiss                     18/JUL/04       8000       28200
       100 Fripp                     10/APR/05       8200       36400
       100 Zlotkey                   29/JAN/08      10500       46900
       100 Raphaely                  07/DEC/02      11000       57900
       100 Cambrault                 15/OCT/07      11000       68900
       100 Errazuriz                 10/MAR/05      12000       80900
```

## ROWS

`rows` uses the value of the current row as the basis for what's included in the function call. Unlike `range`, if the value specified in the `order by` clause has multiple values, it won't include that whole set of values.

```sql
SELECT
    manager_id,
    last_name,
    salary,
    SUM(salary) OVER (ORDER BY salary rows UNBOUNDED PRECEDING) cume_salary
FROM employees
where manager_id = 100
order by salary
```
Output:
```
MANAGER_ID LAST_NAME                     SALARY CUME_SALARY
---------- ------------------------- ---------- -----------
       100 Mourgos                         5800        5800
       100 Vollman                         6500       12300
       100 Kaufling                        7900       20200
       100 Weiss                           8000       28200
       100 Fripp                           8200       36400
       100 Zlotkey                        10500       46900
       100 Cambrault                      11000       57900
       100 Raphaely                       11000       68900
       100 Errazuriz                      12000       80900
```

As you can see, it is cumulative based on the current row. For Cambrault, it includes their salary (11000), and all rows that return before. Similarly for Raphaely, it includes their salary (11000) and all previous rows, including those with the same salary.

## BETWEEN

When defining the rows or range, you can specify `between` to control the start and and end row/range of the window. Usually, you will define a row/range in relation to the current row using the `preceding` or `following` keyword, but you can also refer to the current row with the keyword `current row`.

When specifying the start row/range, you can make use of the `preceding` keyword to define the start row/range - usually in relation to the current row. The syntax is _expression_ `preceding`.

When specifying the end row/range, you can make use of the `following` keyword to define the end row/range - usually in relation to the current row. The syntax is _expression_ `following`.

If using the `rows` or `range` clause,  _expression_ can be a constant or an expression that evaluates to a numeric value. Additionally, if using the `range` clause, _expression_ can be an interval. You can also use the `unbounded` keyword as the _expression_ to refer to all rows.

As an example to include all rows in the window, you would do `rows between unbounded preceding and unbounded following` as demonstrated below.

```sql
SELECT
    manager_id,
    last_name,
    salary,
    SUM(salary) OVER (ORDER BY salary rows between unbounded PRECEDING and unbounded following) sum_sal
FROM employees
order by salary
```
Output:
```
MANAGER_ID LAST_NAME                     SALARY    SUM_SAL
---------- ------------------------- ---------- ----------
       121 Olson                           2100     691416
       120 Markle                          2200     691416
       122 Philtanker                      2200     691416

 107 rows selected
```

If you leave off the `between` keyword, the statement becomes the starting point and the current row the end point.

```sql
SELECT
    manager_id,
    last_name,
    salary,
    SUM(salary) OVER (ORDER BY salary rows 1 PRECEDING) cume_salary
FROM employees
order by salary
```
equivelant to:
```sql
SELECT
    manager_id,
    last_name,
    salary,
    SUM(salary) OVER (ORDER BY salary rows between 1 PRECEDING and current row) cume_salary
FROM employees
order by salary
```
Output:
```
MANAGER_ID LAST_NAME                     SALARY CUME_SALARY
---------- ------------------------- ---------- -----------
       121 Olson                           2100        2100
       120 Markle                          2200        4300
       122 Philtanker                      2200        4400
       120 Landry                          2400        4600

 107 rows selected
```
