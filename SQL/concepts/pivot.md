# PIVOT

The `pivot` clause on an SQL query allows you to transpose column values into column headers, using an aggregate function.

It requires you to pass in an predefined set of columns that you want to be transposed into a header, by using the clause `aggregate for column in (list)`

```sql
select *
from (
    select job_id, employee_id
    from employees
)
pivot (
    count(employee_id) for job_id in (
        'AC_ACCOUNT' AC_ACCOUNT,
        'AC_MGR' AC_MGR,
        'AD_ASST' AD_ASST,
        'AD_PRES' AD_PRES,
        'AD_VP' AD_VP,
        'FI_ACCOUNT' FI_ACCOUNT,
        'FI_MGR' FI_MGR,
        'HR_REP' HR_REP,
        'IT_PROG' IT_PROG,
        'MK_MAN' MK_MAN,
        'MK_REP' MK_REP,
        'PR_REP' PR_REP,
        'PU_CLERK' PU_CLERK,
        'PU_MAN' PU_MAN,
        'SA_MAN' SA_MAN,
        'SA_REP' SA_REP,
        'SH_CLERK' SH_CLERK,
        'ST_CLERK' ST_CLERK
    )
)
```
Output:
```
AC_ACCOUNT     AC_MGR    AD_ASST    AD_PRES      AD_VP FI_ACCOUNT     FI_MGR     HR_REP    IT_PROG     MK_MAN     MK_REP     PR_REP   PU_CLERK     PU_MAN     SA_MAN     SA_REP   SH_CLERK   ST_CLERK
---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ---------- ----------
         1          1          1          1          2          5          1          1          5          1          1          1          5          1          5         30         20         20
```

Looking at the documentation, you would see that you the pivot_in_clause supports the `any` keyword or a subquery. It's worth noting this bit of information:

Any:

> The ANY keyword is used only in conjunction with the XML keyword. The ANY keyword acts as a wildcard and is similar in effect to subquery. The output is not the same cross-tabular format returned by non-XML pivot queries. Instead of multiple columns specified in the pivot_in_clause, the ANY keyword produces a single XML string column. The XML string for each row holds aggregated data corresponding to the implicit GROUP BY value of that row. However, in contrast to the behavior when you specify subquery, the ANY wildcard produces an XML string for each output row that includes only the pivot values found in the input data corresponding to that row.

Subquery:

> A subquery is used only in conjunction with the XML keyword. When you specify a subquery, all values found by the subquery are used for pivoting. The output is not the same cross-tabular format returned by non-XML pivot queries. Instead of multiple columns specified in the pivot_in_clause, the subquery produces a single XML string column. The XML string for each row holds aggregated data corresponding to the implicit GROUP BY value of that row. The XML string for each output row includes all pivot values found by the subquery, even if there are no corresponding rows in the input data.
>  
> The subquery must return a list of unique values at the execution time of the pivot query. If the subquery does not return a unique value, then Oracle Database raises a run-time error. Use the DISTINCT keyword in the subquery if you are not sure the query will return unique values.

```sql
select *
from (
    select job_id, employee_id
    from employees
    where job_id in ('AD_VP','FI_ACCOUNT')
)
pivot xml( count(employee_id) for job_id in (any) )
```
Output:
```xml
<PivotSet>
    <item>
        <column name = "JOB_ID">AD_VP</column>
        <column name = "COUNT(EMPLOYEE_ID)">2</column>
    </item>
    <item>
        <column name = "JOB_ID">FI_ACCOUNT</column>
        <column name = "COUNT(EMPLOYEE_ID)">5</column>
    </item>
</PivotSet>
```
