# AVG

`avg` is an aggregate function, to get the average value amongst a set of rows. This only works on numerican data types.

```sql
select avg(salary) avg_it_sal
from employees
where job_id = 'IT_PROG'
```
Output:
```
AVG_IT_SAL
----------
      5760
```

```sql
select job_id, avg(salary) avg_job_sal
from employees
group by job_id
```

Output:
```
JOB_ID     AVG_JOB_SAL
---------- -----------
IT_PROG           5760
AC_MGR           12008
AC_ACCOUNT        8300
ST_MAN            7280
PU_MAN           11000
AD_ASST           4400
AD_VP            17000
SH_CLERK          3215
FI_ACCOUNT        7920
FI_MGR           12008
PU_CLERK          2780
SA_MAN           12200
MK_MAN           13000
PR_REP           10000
AD_PRES          24000
SA_REP            8350
MK_REP            6000
ST_CLERK          2785
HR_REP            6500

 19 rows selected
```

It can also be used as an analytic function:

```sql
select employee_id
, job_id
, first_name
, last_name
, avg(salary) over (partition by job_id) job_avg_sal
from employees
order by job_id
```

Output:
```
EMPLOYEE_ID JOB_ID     FIRST_NAME           LAST_NAME                 JOB_AVG_SAL
----------- ---------- -------------------- ------------------------- -----------
        206 AC_ACCOUNT William              Gietz                            8300
        205 AC_MGR     Shelley              Higgins                         12008
        200 AD_ASST    Jennifer             Whalen                           4400
        100 AD_PRES    Steven               King                            24000
        102 AD_VP      Lex                  De Haan                         17000
        101 AD_VP      Neena                Kochhar                         17000
        110 FI_ACCOUNT John                 Chen                             7920
        109 FI_ACCOUNT Daniel               Faviet                           7920

107 rows selected
```
