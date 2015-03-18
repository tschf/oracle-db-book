# Modelling

## Intro

SQL statements have a model clause, which allows you to perform calculations, across rows, for the data that has been returned from your SQL query.

First, you need to categorise your columns in three areas:

1. partition - defines which columns are used to split up the rows into invidual groups
* dimension - defines which columns are used to identify the rows in each partition
* measure - defines which columns can be used in calculations/re-assignment

Every column that exists in the select clause, must also be present in either partition, dimension or measure attributes.

Then we have a `rules` statement that can be used to perform calculations on the rows (technically, you don't need the `rules` keyword, but it'll be more clear if you leave it there). You reference the column values defined in the `measures` expression by placing an expression in square brackets, for each `dimension` defined, separated by a comma.the dimension column(s) values in the square brackets directly following.

If you pass in a value without referencing the dimension, it is assumed to be an `=` expression for that dimension. In the below example, if I instead had `salary[200]`, that would be the same as `salary[employee_id=200]`.

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

## Adding data

### New rows

The `rules` expression has an option that defines the behaviour when a cell reference does not yet exist. The three options you can specify are:

1. UPSERT
2. UPSERT ALL
3. UPDATE

`upsert` is the implied value if nothing is specified. It creates the cell if the dimensions you pass in do not result in finding any existing cells, but only if the lookup value is a constant value (no wildcards/loops) on a `positional reference` (not referring to the dimension column - known as a `symbolic reference`).

`upsert all` much like upsert, but will also insert rows when using using range checks such as any, in, etc.

`update` will only update values for existing cells. Any reference to a cell that does not exist will simply be ignored.


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
    rules upsert (
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

### New columns

You can define aliases in the `measures` clause, then can then be referred to in the original select list. Using the increased salary idea, instead of setting the original value, we could define new_salary like so:

```sql
select
    employee_id,
    first_name,
    salary,
    new_Salary
from employees
model
    return updated rows
    dimension by (employee_id)
    measures (first_name, salary, salary new_Salary)
    rules (
        new_salary[employee_id >= 200] = salary[cv(employee_id)] * 1.1

    )
order by employee_id
```

Output:
```
EMPLOYEE_ID FIRST_NAME               SALARY NEW_SALARY
----------- -------------------- ---------- ----------
        200 Jennifer                   4400       4840
        201 Michael                   13000      14300
        202 Pat                        6000       6600
        203 Susan                      6500       7150
        204 Hermann                   10000      11000
        205 Shelley                   12008    13208.8
        206 William                    8300       9130

 7 rows selected

```
## Multiple dimensions

You can specify multiple dimension columns, in order to identify rows by two conditions. Expanding on the example from the beginning of the document, in addition to employees with an employee_id greater than or equal to 200, you may also like to restrict it to those hired on or before 17-SEP-2003.

```sql
select
    employee_id,
    first_name,
    hire_date,
    salary
from employees
model
    return updated rows
    dimension by (employee_id, hire_date)
    measures (first_name, salary)
    rules (
        salary[employee_id >= 200,hire_date <= to_date('17-SEP-03', 'DD-MON-YY')] = salary[cv(employee_id),cv(hire_date)] * 1.1

    )
order by employee_id
```
Output:
```
EMPLOYEE_ID FIRST_NAME           HIRE_DATE     SALARY
----------- -------------------- --------- ----------
        200 Jennifer             17/SEP/03       4840
        203 Susan                07/JUN/02       7150
        204 Hermann              07/JUN/02      11000
        205 Shelley              07/JUN/02    13208.8
        206 William              07/JUN/02       9130
```

So, you separate each measure by a comma in the measure lookup. If, you have multiple dimensions, you can use the `any` keyword as a wild card to refer to all possible values.

## Aggregate functions

The use of aggregate functions such as `sum` are available for us to use. In this case, you still need to use cell references to determine which cell values are used in the function.

Use of `any` on a right-hand lookup will not restrict to the rows matched on a lef-hand lookup. So you would likely need to re-apply the filter as shown:

```sql
select
    employee_id,
    first_name,
    salary,
    hire_date,
    running_salary,
    total_Salary
from employees
model
    return updated rows
    dimension by (employee_id, hire_date)
    measures (first_name, salary, 0 total_Salary, 0 running_salary)
    rules (
        total_salary[employee_id >= 200,any] = sum(salary)[employee_id >= 200, any],
        running_salary[employee_id >= 200,any] = sum(salary)[employee_id >= 200, hire_date < cv(hire_date)]

    )
order by hire_date
```

Output:
```
EMPLOYEE_ID FIRST_NAME               SALARY HIRE_DATE RUNNING_SALARY TOTAL_SALARY
----------- -------------------- ---------- --------- -------------- ------------
        203 Susan                      6500 07/JUN/02                       60208
        205 Shelley                   12008 07/JUN/02                       60208
        204 Hermann                   10000 07/JUN/02                       60208
        206 William                    8300 07/JUN/02                       60208
        200 Jennifer                   4400 17/SEP/03          36808        60208
        201 Michael                   13000 17/FEB/04          41208        60208
        202 Pat                        6000 17/AUG/05          54208        60208

 7 rows selected
```

This example is keeping a running total of the salary over time by only applying sum to values in hires before today.

## Lookup Table

As a part of the model, you can also create sub-models based on an entirely separate query, which can then be referenced in your main model. These are referred to as a `reference model`.

Suppose we want to get the location_id of the staff member's department:

```sql
select
    employee_id,
    first_name,
    salary,
    hire_date,
    department_id,
    loc
from employees
model
    return updated rows
    reference depts on (select department_id, location_id from departments)
        dimension by (department_id)
        measures (location_id)
    main emps
        dimension by (employee_id, hire_date)
        measures (first_name, salary, department_id, 0 loc)
        rules (

            loc[employee_id >= 200, any] = depts.location_id[department_id[cv(employee_id),cv(hire_date)]]

        )
order by employee_id
```

Output:
```
EMPLOYEE_ID FIRST_NAME               SALARY HIRE_DATE DEPARTMENT_ID        LOC
----------- -------------------- ---------- --------- ------------- ----------
        200 Jennifer                   4400 17/SEP/03            10       1700
        201 Michael                   13000 17/FEB/04            20       1800
        202 Pat                        6000 17/AUG/05            20       1800
        203 Susan                      6500 07/JUN/02            40       2400
        204 Hermann                   10000 07/JUN/02            70       2700
        205 Shelley                   12008 07/JUN/02           110       1700
        206 William                    8300 07/JUN/02           110       1700

 7 rows selected

```

## Data range

### For Loops

`for` loops can be used on the left hand side of an assignment. The syntax is: `for dimension in list`. List can be a constant list such as: `(1,2,3); ('string1', 'string2', 'string3')` or a subquery.

Range examples:

```
for employee_id in (select employee_id from employees where ...)
for employee_id in (103,104,105,106,107)
for employee_id from 103 to 107 increment 1
for number_in_string like 'Version%' from 1 to 3 increment 1
--above produces 'Version1', 'Version2', 'Version3'
```


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

        first_name[for employee_id in (select employee_id from employees where job_id = 'IT_PROG')] = 'Programmer-' || first_name[cv()]

    )
order by employee_id
```

Output:
```
EMPLOYEE_ID FIRST_NAME               SALARY
----------- -------------------- ----------
        103 Programmer-Alexander       9000
        104 Programmer-Bruce           6000
        105 Programmer-David           4800
        106 Programmer-Valli           4800
        107 Programmer-Diana           4200
```

Right hand side loop:


### Iteration models

The `iterate` clause allows you to repeat an assignment, with an optional `until` clause. The rule will repeat until either there are no more iterations left or the `until` condition is met.

```sql
select
    employee_id,
    first_name,
    salary,
    new_Salary
from employees
where employee_id >= 200
model
    return updated rows
    dimension by (employee_id)
    measures (first_name, salary, salary new_salary)
    rules iterate(3)(

        new_salary[any] = new_salary[cv()]*(ITERATION_NUMBER+1)
    )
order by employee_id
```
Output:
```
EMPLOYEE_ID FIRST_NAME               SALARY NEW_SALARY
----------- -------------------- ---------- ----------
        200 Jennifer                   4400      26400
        201 Michael                   13000      78000
        202 Pat                        6000      36000
        203 Susan                      6500      39000
        204 Hermann                   10000      60000
        205 Shelley                   12008      72048
        206 William                    8300      49800

 7 rows selected
```

## Useful resources

[Model clause syntax](http://docs.oracle.com/cd/E11882_01/server.112/e41084/statements_10002.htm#SQLRF01702)  
[Model Expressions in SQL Language Reference](http://docs.oracle.com/cd/E11882_01/server.112/e41084/expressions010.htm#SQLRF52086)  
[SQL For Modelling in the Data Warehouse Guide](http://docs.oracle.com/cd/B19306_01/server.102/b14223/sqlmodel.htm)  
[SQL Model clause tutorial](http://rwijk.blogspot.nl/2007/10/sql-model-clause-tutorial-part-one.html) (3 part series)
