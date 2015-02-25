# GROUPING

Oracle documentation: https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions071.htm#SQLRF00647

Used to distinguish between regular rows and superappgregate rows produces as a result of using a group by extension. If it is a regular row it returns `0` otherwise, it returns `1`

```sql
select
    job_id,
    grouping(job_id) aa,
    grouping_id(job_id) aa,
    count(1)
from employees
group by
    grouping sets (
        (job_id),
        ()
    )
```
Output:
```
JOB_ID       COUNT(1)
---------- ----------
AC_ACCOUNT          1
AC_MGR              1
AD_ASST             1
AD_PRES             1
AD_VP               2
FI_ACCOUNT          5
FI_MGR              1
HR_REP              1
IT_PROG             5
MK_MAN              1
MK_REP              1
PR_REP              1
PU_CLERK            5
PU_MAN              1
SA_MAN              5
SA_REP             30
SH_CLERK           20
ST_CLERK           20
ST_MAN              5
ALL JOBS          107

 20 rows selected
```
