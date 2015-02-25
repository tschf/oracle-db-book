# GROUPING_ID

Oracle documentation: https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions072.htm#SQLRF00648

`grouping_id` returning the `grouping` bit vector. It is especially useful to determine the group by level.

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
  , sum(sale_amt)
from sales
group by cube (dept, position_id)
order by dept, position_id
```
Output:
```
DEPT POSITION_ID GROUPING_ID(DEPT,POSITION_ID) SUM(SALE_AMT)
---- ----------- ----------------------------- -------------
10   AC_ACCOUNT                              0          1300
10   MK_REP                                  0           900
10   SA_REP                                  0          2100
10                                           1          4300
20   MK_MAN                                  0          3100
20                                           1          3100
30   PU_MAN                                  0          1100
30   SA_MAN                                  0           899
30   SA_REP                                  0          1590
30                                           1          3589
50   AC_MGR                                  0          2499
50                                           1          2499
     AC_ACCOUNT                              2          1300
     AC_MGR                                  2          2499
     MK_MAN                                  2          3100
     MK_REP                                  2           900
     PU_MAN                                  2          1100
     SA_MAN                                  2           899
     SA_REP                                  2          3690
                                             3         13488

 20 rows selected
```

Given the above, we can easily identify what level of summary information is being provided with the grouping_id column.

It may also be useful to review the asktom article on the topic: https://asktom.oracle.com/pls/asktom/f?p=100:11:::::P11_QUESTION_ID:37355353762363
