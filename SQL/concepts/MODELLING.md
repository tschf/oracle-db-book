# MODELLING

## Intro

SQL statements have a model clause, which allows you to perform calculations, across rows, for the data that has been returned from your SQL query.

First, you need to categorise your columns in three areas:

1. partition - defines which columns are used to split up the rows into invidual groups
* dimension - defines which columns are used to identify the rows in each partition
* measure - defines which columns can be used in calculations/re-assignment

Every column that exists in the select clause, must also be present in either partition, dimension or measure attributes.

Then we have a `rules` statement that can be used to perform calculations on the rows (technically, you don't need the `rules` keyword, but it'll be more clear if you leave it there). You reference the column values defined in the `measures` expression by placing the dimension column(s) values in the square brackets directly following.

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

* The clause `return updated rows` tells us to only bring back rows that have had their values modified. The alternative would be `return all rows` which is the default.
* In the salary assignment, we use the function [`cv`](../functions/CV.md). This gets the current value being processed.

## Adding rows

If you assign a value to a dimension that does no exist, a new row will be created.


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
        salary[employee_id >= 200] = salary[cv(employee_id)] * 1.1,
        salary[250] = salary[cv(employee_id)],
        first_name[250] = 'NO-NAME'
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
        250 NO-NAME

 8 rows selected
```

As you can see, using `cv` here (for salary) simply returns NULL for the non-existent row.

In the example above, we aren't using the `partition by` clause, but if we were, that row we added would be looked up (and consequently added) for each partition.

## Useful resources

[Model clause syntax](http://docs.oracle.com/cd/E11882_01/server.112/e41084/statements_10002.htm#SQLRF01702)  
[Model Expressions in SQL Language Reference](http://docs.oracle.com/cd/E11882_01/server.112/e41084/expressions010.htm#SQLRF52086)  
[SQL For Modelling in the Data Warehouse Guide](http://docs.oracle.com/cd/B19306_01/server.102/b14223/sqlmodel.htm)  
[SQL Model clause tutorial part 1](http://rwijk.blogspot.nl/2007/10/sql-model-clause-tutorial-part-one.html)  
[SQL Model clause tutorial part 2](http://rwijk.blogspot.nl/2007/10/sql-model-clause-tutorial-part-two.html)  
[SQL Model clause tutorial part 3](http://rwijk.blogspot.nl/2009/01/sql-model-clause-tutorial-part-three.html)  
