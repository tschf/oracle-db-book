# MODELLING

## Intro

SQL statements have a model clause, which allows you to perform calculations, across rows, for the data that has been returned from your SQL query.

First, you need to categorise your columns in three areas:

1. partition - defines which columns are used to split up the rows into invidual groups
* dimension - defines which columns are used to identify the rows in each partition
* measure - defines which columns can be used in calculations/re-assignment

Every column that exists in the select clause, must also be present in either partition, dimension or measure attributes.

Then we have a rules statement that can be used to perform calculations on the rows. You reference the column values defined in the `measures` expression by placing the dimension column(s) values in the square brackets directly following.

The dimension expression doesn't need to be an exact value - you can use expressions.

As a basic example, you may want to increase any employee's salary by 10% if the employee_id is greater than or equal to 200.

```sql
select
    employee_id,
    first_name,
    salary
from employees
model
    return updated rows
    dimension by (employee_id)
    measures (first_name, salary)
    rules (
        salary[employee_id >= 200] = salary[cv(employee_id)] * 1.1
    )
order by employee_id
```
Output:
```
EMPLOYEE_ID FIRST_NAME               SALARY
----------- -------------------- ----------
        200 Jennifer                   4840
        201 Michael                   14300
        202 Pat                        6600
        203 Susan                      7150
        204 Hermann                   11000
        205 Shelley                 13208.8
        206 William                    9130

 7 rows selected
```

A couple of concepts not yet talked about:

* The clause `return updated rows` tells us to only bring back rows that have had their values modified  
* In the salary assignment, we use the function [`cv`](../functions/CV.md). This gets the current value being processed.
