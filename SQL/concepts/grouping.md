# Grouping

## Basics

Sometimes the results of your SQL query, will contain duplicates.

```sql
select first_name, job_id
from employees
where job_id = 'SA_REP'
and first_name = 'David'
```

```
FIRST_NAME           JOB_ID
-------------------- ----------
David                SA_REP
David                SA_REP
```

There are a couple of ways to avoid this.

One is to use the `distinct` keyword before your column specification in the `select` portion of your query.

```sql
select distinct first_name, job_id
from employees
where job_id = 'SA_REP'
and first_name = 'David'
```

or add the `group by` clause to your query.

```sql
select first_name, job_id
from employees
where job_id = 'SA_REP'
and first_name = 'David'
group by first_name, job_id
```
Output:
```
FIRST_NAME           JOB_ID
-------------------- ----------
David                SA_REP  
```

## Aggregation

The other reason (perhaps the main reason) to use the `group by` clause in your query is that you want to use one of the number of aggregate functions available. One common one is `count` - for each group, you may want to know the total number of rows that would have been.

```sql
select first_name, job_id, count(first_name)
from employees
group by first_name, job_id
order by count(first_name) desc
```
Output:
```
FIRST_NAME           JOB_ID     COUNT(FIRST_NAME)
-------------------- ---------- -----------------
David                SA_REP                     2
James                ST_CLERK                   2
Peter                SA_REP                     2
Lex                  AD_VP                      1
Jennifer             SH_CLERK                   1
Danielle             SA_REP                     1

 104 rows selected
```

Excluding any aggregate function calls you make, if you specify a `group by` clause, at minimum, the column specification of the `group by` clause should match that of the `select` statement.

You can also add columns to the `group by` clause that aren't specifically mentioned in the `select` statement, but are available in any of the tables/views you are querying.

In the employee table, there are three Davids overall. Just querying the first_name, we can get David on two distinct rows by adding job_id to the `group by` clause.

```sql
select first_name, count(1)
from employees
where first_name = 'David'
group by first_name, job_id
```
Output:
```
FIRST_NAME             COUNT(1)
-------------------- ----------
David                         2
David                         1
```
### Function List

For a full list of all available aggregate functions, refer to the official documentation: http://docs.oracle.com/cd/E11882_01/server.112/e41084/functions003.htm#SQLRF20035



## Group by extension

In addition to regular groupings, you can also produce superaggregate groupings bu using the `rollup` or `cube` functions in the `group by` clause.

### Rollup

The best explanation for the `rollup` extension is explained on the [SQL for Aggregation in Data Warehouses ](http://docs.oracle.com/cd/B19306_01/server.102/b14223/aggreg.htm#i1007413) guide:

> The action of `ROLLUP` is straightforward: it creates subtotals that roll up from the most detailed level to a grand total, following a grouping list specified in the `ROLLUP` clause. `ROLLUP` takes as its argument an ordered list of grouping columns. First, it calculates the standard aggregate values specified in the GROUP BY clause. Then, it creates progressively higher-level subtotals, moving from right to left through the list of grouping columns. Finally, it creates a grand total.
>
> ROLLUP creates subtotals at n+1 levels, where n is the number of grouping columns. For instance, if a query specifies ROLLUP on grouping columns of time, region, and department(n=3), the result set will include rows at four aggregation levels.

The `rollup` function create summary rows for each grouping, as well as a grand total row.

Given the following sales data:

```
DEPT POSITION_ID CLASSIFICATION   SALE_AMT
---- ----------- -------------- ----------
10   AC_ACCOUNT               2       1300
10   MK_REP                   2        900
10   SA_REP                   1       1200
10   SA_REP                   2        900
20   MK_MAN                   1       1800
20   MK_MAN                   2       1300
30   PU_MAN                   1       1100
30   SA_MAN                   2        899
30   SA_REP                   1        640
30   SA_REP                   2        950
50   AC_MGR                   1       1200
50   AC_MGR                   2       1299
```

If we just group by one column, we'll only get one summary row - which will be the grand total:

```sql
select dept, sum(sale_amt)
from sales
group by rollup(dept)
order by dept
```
Output:
```
DEPT SUM(SALE_AMT)
---- -------------
10            4300
20            3100
30            3589
50            2499
-            13488
```

If there is more than one column in the `rollup`, going from right to left, there will be a summary row for each of the columns to the left. For example, if we have `dept, position_id`, there will be a summary row each department.

```sql
select dept, position_id, sum(sale_amt)
from sales
group by rollup(dept, position_id)
order by dept, position_id
```
Output:
```
DEPT POSITION_ID SUM(SALE_AMT)
---- ----------- -------------
10   AC_ACCOUNT           1300
10   MK_REP                900
10   SA_REP               2100
10   -                    4300
20   MK_MAN               3100
20   -                    3100
30   PU_MAN               1100
30   SA_MAN                899
30   SA_REP               1590
30   -                    3589
50   AC_MGR               2499
50   -                    2499
-    -                   13488

 13 rows selected
```

If we have three column expressions in the rollup function dept, position_id and classification, we will have 2 levels of summary rows:

1. Subtotals for classification across each group of dept and position_id
* Subtotals for position_id and classification for each dept

```sql
select dept, position_id, classification, sum(sale_amt), count(dept)
from sales
group by rollup(dept, position_id, classification)
order by dept, position_id, classification
```
Output:
```
DEPT POSITION_ID CLASSIFICATION SUM(SALE_AMT) COUNT(DEPT)
---- ----------- -------------- ------------- -----------
10   AC_ACCOUNT               2          1300           1
10   AC_ACCOUNT  -                       1300           1
10   MK_REP                   2           900           1
10   MK_REP      -                        900           1
10   SA_REP                   1          1200           1
10   SA_REP                   2           900           1
10   SA_REP      -                       2100           2
10   -           -                       4300           4
20   MK_MAN                   1          1800           1
20   MK_MAN                   2          1300           1
20   MK_MAN      -                       3100           2
20   -           -                       3100           2
30   PU_MAN                   1          1100           1
30   PU_MAN      -                       1100           1
30   SA_MAN                   2           899           1
30   SA_MAN      -                        899           1
30   SA_REP                   1           640           1
30   SA_REP                   2           950           1
30   SA_REP      -                       1590           2
30   -           -                       3589           4
50   AC_MGR                   1          1200           1
50   AC_MGR                   2          1299           1
50   AC_MGR      -                       2499           2
50   -           -                       2499           2
-    -           -                      13488          12

 25 rows selected
```

You can also do a partial `rollup`. Given the last example, we can move the `rollup` to be for the second two column expressions only, which would result in avoiding the grand total row.

Giving us sub-totals at:

* dept, position_id, classification
* dept, position_id
* dept

```sql
select dept, position_id, classification, sum(sale_amt)
from sales
group by dept, rollup(position_id, classification)
order by dept, position_id
```
Output:
```
DEPT POSITION_ID CLASSIFICATION SUM(SALE_AMT)
---- ----------- -------------- -------------
10   AC_ACCOUNT               2          1300
10   AC_ACCOUNT                          1300
10   MK_REP                   2           900
10   MK_REP                               900
10   SA_REP                   1          1200
10   SA_REP                   2           900
10   SA_REP                              2100
10                                       4300
20   MK_MAN                   1          1800
20   MK_MAN                   2          1300
20   MK_MAN                              3100
20                                       3100
30   PU_MAN                   1          1100
30   PU_MAN                              1100
30   SA_MAN                   2           899
30   SA_MAN                               899
30   SA_REP                   1           640
30   SA_REP                   2           950
30   SA_REP                              1590
30                                       3589
50   AC_MGR                   1          1200
50   AC_MGR                   2          1299
50   AC_MGR                              2499
50                                       2499

 24 rows selected
```


### Cube

> CUBE takes a specified set of grouping columns and creates subtotals for all of their possible combinations.

In a two-dimensional `cube`, we get a summary row for each department. Then we get a summary row for each `position_id`. That gives us all possible subtotals from the cube column expression list.

```sql
select dept, position_id, sum(sale_amt)
from sales
group by cube(dept, position_id)
order by dept, position_id
```
Output:
```
DEPT POSITION_ID SUM(SALE_AMT)
---- ----------- -------------
10   AC_ACCOUNT           1300
10   MK_REP                900
10   SA_REP               2100
10                        4300
20   MK_MAN               3100
20                        3100
30   PU_MAN               1100
30   SA_MAN                899
30   SA_REP               1590
30                        3589
50   AC_MGR               2499
50                        2499
     AC_ACCOUNT           1300
     AC_MGR               2499
     MK_MAN               3100
     MK_REP                900
     PU_MAN               1100
     SA_MAN                899
     SA_REP               3690
                         13488

 20 rows selected

```

Adding classification to the query, at the first level we get:

* each classification (position_id and dept being NULL)
* each position_id (dept and classification being NULL)
* each dept (position_id and classification being NULL)

Then at the second level:

* each classification and position_id (dept being NULL)
* each classification and dept (position_id being NULL)
* each position_id and dept (classification being NULL)

```sql
select dept, position_id, classification, sum(sale_amt)
from sales
group by cube(dept, position_id, classification)
order by dept, position_id, classification
```
Output:
```
DEPT POSITION_ID CLASSIFICATION SUM(SALE_AMT)
---- ----------- -------------- -------------
10   AC_ACCOUNT               2          1300
10   AC_ACCOUNT                          1300
10   MK_REP                   2           900
10   MK_REP                               900
10   SA_REP                   1          1200
10   SA_REP                   2           900
10   SA_REP                              2100
10                            1          1200
10                            2          3100
10                                       4300
20   MK_MAN                   1          1800
20   MK_MAN                   2          1300
20   MK_MAN                              3100
20                            1          1800
20                            2          1300
20                                       3100
30   PU_MAN                   1          1100
30   PU_MAN                              1100
30   SA_MAN                   2           899
30   SA_MAN                               899
30   SA_REP                   1           640
30   SA_REP                   2           950
30   SA_REP                              1590
30                            1          1740
30                            2          1849
30                                       3589
50   AC_MGR                   1          1200
50   AC_MGR                   2          1299
50   AC_MGR                              2499
50                            1          1200
50                            2          1299
50                                       2499
     AC_ACCOUNT               2          1300
     AC_ACCOUNT                          1300
     AC_MGR                   1          1200
     AC_MGR                   2          1299
     AC_MGR                              2499
     MK_MAN                   1          1800
     MK_MAN                   2          1300
     MK_MAN                              3100
     MK_REP                   2           900
     MK_REP                               900
     PU_MAN                   1          1100
     PU_MAN                              1100
     SA_MAN                   2           899
     SA_MAN                               899
     SA_REP                   1          1840
     SA_REP                   2          1850
     SA_REP                              3690
                              1          5940
                              2          7548
                                        13488

 52 rows selected
```
We can use a partial cube to limit the superaggregate rows returned. If we cube the second two columns, we get:

* each dept (position_id and classification being NULL)

Then at the second level:

* each dept and position_id (classification being NULL)
* each position_id and classification (position_id being NULL)

So, using the partial cube, we have eliminated summary rows for classification and possition_id, as well as the grand total.

```sql
select dept, position_id, classification, sum(sale_amt)
from sales
group by dept, cube(position_id, classification)
order by dept, position_id, classification
```
Output:
```
DEPT POSITION_ID CLASSIFICATION SUM(SALE_AMT)
---- ----------- -------------- -------------
10   AC_ACCOUNT               2          1300
10   AC_ACCOUNT                          1300
10   MK_REP                   2           900
10   MK_REP                               900
10   SA_REP                   1          1200
10   SA_REP                   2           900
10   SA_REP                              2100
10                            1          1200
10                            2          3100
10                                       4300
20   MK_MAN                   1          1800
20   MK_MAN                   2          1300
20   MK_MAN                              3100
20                            1          1800
20                            2          1300
20                                       3100
30   PU_MAN                   1          1100
30   PU_MAN                              1100
30   SA_MAN                   2           899
30   SA_MAN                               899
30   SA_REP                   1           640
30   SA_REP                   2           950
30   SA_REP                              1590
30                            1          1740
30                            2          1849
30                                       3589
50   AC_MGR                   1          1200
50   AC_MGR                   2          1299
50   AC_MGR                              2499
50                            1          1200
50                            2          1299
50                                       2499

 32 rows selected
```

### Grouping sets

If you don't want to get every possible combination of summary rows as is the case with cube, you can use the `grouping sets` expression.

With this, you can specify multiple groupings. It then combines the results from each grouping by using a `UNION ALL`.

The first set should include the column list as per a regular `group by` expression, then each additional set can be a subset of that to determine the appropriate superaggregate row.

As a basic example, we may want a summary row for each department, we can add a set with just `(dept)` like so:

```sql
select dept, position_id, classification, sum(sale_amt)
from sales
group by
    grouping sets (
        (dept, position_id, classification),
        (dept)
    )
order by dept, position_id, classification
```
Output:
```
DEPT POSITION_ID CLASSIFICATION SUM(SALE_AMT)
---- ----------- -------------- -------------
10   AC_ACCOUNT               2          1300
10   MK_REP                   2           900
10   SA_REP                   1          1200
10   SA_REP                   2           900
10                                       4300
20   MK_MAN                   1          1800
20   MK_MAN                   2          1300
20                                       3100
30   PU_MAN                   1          1100
30   SA_MAN                   2           899
30   SA_REP                   1           640
30   SA_REP                   2           950
30                                       3589
50   AC_MGR                   1          1200
50   AC_MGR                   2          1299
50                                       2499

 16 rows selected

```
And to get the total row, we pass in the empty set:

```sql
select dept, position_id, classification, sum(sale_amt)
from sales
group by
    grouping sets (
        (dept, position_id, classification),
        (dept),
        ()
    )
order by dept, position_id, classification
```
Output:
```
DEPT POSITION_ID CLASSIFICATION SUM(SALE_AMT)
---- ----------- -------------- -------------
10   AC_ACCOUNT               2          1300
10   MK_REP                   2           900
10   SA_REP                   1          1200
10   SA_REP                   2           900
10                                       4300
20   MK_MAN                   1          1800
20   MK_MAN                   2          1300
20                                       3100
30   PU_MAN                   1          1100
30   SA_MAN                   2           899
30   SA_REP                   1           640
30   SA_REP                   2           950
30                                       3589
50   AC_MGR                   1          1200
50   AC_MGR                   2          1299
50                                       2499
                                        13488

 17 rows selected
```
