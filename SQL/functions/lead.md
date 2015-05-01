# LEAD

LEAD is a way to access another row of the table you are querying without actually performing a self join.

Imagine the use case you want to do a report on employees, and for each employee you list, you also want to know the employee with the next highest salary.

Without the use of `LEAD` you could do something like:

```sql
with emps as (
    select 1 ID, 'John' first, 'Smith' last, 3300 sal from dual union all
    select 2 ID,'Peter' first, 'Rogers' last, 9000 sal from dual union all
    select 3 ID,'Deb' first, 'Harrison' last, 5500 sal from dual union all
    select 4 ID,'Slade' first, 'Wilson' last, 343 sal from dual union all
    select 5 ID,'Martha' first, 'Stewart' last, 1100 sal from dual
),
emp_sal_ranked as (
    select
        emp1.first
      , emp1.last
      , emp1.sal
      , emp2.first next_first
      , emp2.last next_last
      , emp2.sal next_sal
      , row_number() over (partition by emp1.ID order by emp2.sal) next_sal_ranked
    from emps emp1, emps emp2
    where emp2.sal (+) > emp1.sal
)
select first, last, sal, next_first, next_last, next_Sal
from emp_sal_ranked
where next_sal_ranked = 1
order by sal
```

```
FIRST  LAST            SAL NEXT_FIRST NEXT_LAST   NEXT_SAL
------ -------- ---------- ---------- --------- ----------
Slade  Wilson          343 Martha     Stewart         1100
Martha Stewart        1100 John       Smith           3300
John   Smith          3300 Deb        Harrison        5500
Deb    Harrison       5500 Peter      Rogers          9000
Peter  Rogers         9000
```

Now we can avoid that self join like so:

```sql
with emps as (
    select 1 ID, 'John' first, 'Smith' last, 3300 sal from dual union all
    select 2 ID,'Peter' first, 'Rogers' last, 9000 sal from dual union all
    select 3 ID,'Deb' first, 'Harrison' last, 5500 sal from dual union all
    select 4 ID,'Slade' first, 'Wilson' last, 343 sal from dual union all
    select 5 ID,'Martha' first, 'Steward' last, 1100 sal from dual
)
select
    first
  , last
  , sal
  , lead(first) over (order by sal) next_first
  , lead(last) over (order by sal) next_last
  , lead(sal) over (order by sal) next_sal
from emps
```
```
FIRST  LAST            SAL NEXT_FIRST NEXT_LAST   NEXT_SAL
------ -------- ---------- ---------- --------- ----------
Slade  Wilson          343 Martha     Steward         1100
Martha Steward        1100 John       Smith           3300
John   Smith          3300 Deb        Harrison        5500
Deb    Harrison       5500 Peter      Rogers          9000
Peter  Rogers         9000
```
Producing a lot cleaner query.

In the above example, I just passed in one parameter to the lead function, but it accepts:

* Column - the value to be returned
* Offset - how many rows to offset by, given the order by clause. The defaults to 1, if left off
* Default - what to return in the case of NULL

You can also respect (default) or ignore NULLs in the test with the syntax: `lead(column IGNORE NULLS)` or `lead(column) IGNORE NULLS`.

Because we are just ordering by sal in the lead call, if two rows have the same sal, it won't get the next highest as my first example does, but the next row in the result set. E.g:

```sql
with emps as (
    select 1 ID, 'John' first, 'Smith' last, 3300 sal from dual union all
    select 2 ID,'Peter' first, 'Rogers' last, 9000 sal from dual union all
    select 3 ID,'Deb' first, 'Harrison' last, 5500 sal from dual union all
    select 4 ID,'Slade' first, 'Wilson' last, 343 sal from dual union all
    select 5 ID,'Martha' first, 'Steward' last, 1100 sal from dual union all
    select 6 ID, 'Bruce' first, 'Wayne' last, 3300 sal from dual
)
select
    first
  , last
  , sal
  , lead(first) over (order by sal) next_first
  , lead(last) over (order by sal) next_last
  , lead(sal) over (order by sal) next_sal
from emps
```
```
FIRST  LAST            SAL NEXT_FIRST NEXT_LAST   NEXT_SAL
------ -------- ---------- ---------- --------- ----------
Slade  Wilson          343 Martha     Steward         1100
Martha Steward        1100 John       Smith           3300
John   Smith          3300 Bruce      Wayne           3300
Bruce  Wayne          3300 Deb        Harrison        5500
Deb    Harrison       5500 Peter      Rogers          9000
Peter  Rogers         9000
```

You may also be interested in the [LAG](LAG.md) function which has the reverse effect of getting the previous row.
