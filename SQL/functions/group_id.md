# GROUP_ID

If you have a complex group by expression, you may get duplicate rows. `group_id` is used to identify duplicates by returning values greater than zero when there is more than one occurrence of that row.

```sql
with sales as
(
select '10' DEPT, 'SA_REP'      POSITION_ID, 1200   sale_amt from dual union all
select '10' DEPT, 'AC_ACCOUNT'  POSITION_ID, 1300   sale_amt from dual union all
select '10' DEPT, 'SA_REP'      POSITION_ID, 900    sale_amt from dual union all
select '10' DEPT, 'MK_REP'      POSITION_ID, 900    sale_amt from dual union all
select '20' DEPT, 'MK_MAN'      POSITION_ID, 1800   sale_amt from dual union all
select '20' DEPT, 'MK_MAN'      POSITION_ID, 1300   sale_amt from dual union all
select '30' DEPT, 'SA_REP'      POSITION_ID, 640    sale_amt from dual union all
select '30' DEPT, 'SA_REP'      POSITION_ID, 950    sale_amt from dual union all
select '30' DEPT, 'PU_MAN'      POSITION_ID, 1100   sale_amt from dual union all
select '30' DEPT, 'SA_MAN'      POSITION_ID, 899    sale_amt from dual union all
select '50' DEPT, 'AC_MGR'      POSITION_ID, 1200   sale_amt from dual union all
select '50' DEPT, 'AC_MGR'      POSITION_ID, 1299   sale_amt from dual
)
select
    dept
  , position_id
  , grouping_id(dept, position_id)
  , group_id()
  , sum(sale_amt)
from sales
group by
    grouping sets (
        (dept, position_id),
        rollup (dept, position_id)
    )
order by dept, position_id
```
Output:
```
DEPT POSITION_ID GROUPING_ID(DEPT,POSITION_ID) GROUP_ID() SUM(SALE_AMT)
---- ----------- ----------------------------- ---------- -------------
10   AC_ACCOUNT                              0          1          1300
10   AC_ACCOUNT                              0          0          1300
10   MK_REP                                  0          0           900
10   MK_REP                                  0          1           900
10   SA_REP                                  0          0          2100
10   SA_REP                                  0          1          2100
10                                           1          0          4300
20   MK_MAN                                  0          0          3100
20   MK_MAN                                  0          1          3100
20                                           1          0          3100
30   PU_MAN                                  0          0          1100
30   PU_MAN                                  0          1          1100
30   SA_MAN                                  0          1           899
30   SA_MAN                                  0          0           899
30   SA_REP                                  0          1          1590
30   SA_REP                                  0          0          1590
30                                           1          0          3589
50   AC_MGR                                  0          1          2499
50   AC_MGR                                  0          0          2499
50                                           1          0          2499
                                             3          0         13488

 21 rows selected
```

We can see here that there are two versions of a lot of the rows, so group_id() has returned 1 for the second occurrence.
