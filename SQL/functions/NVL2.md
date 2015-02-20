# NVL2

Oracle documentation: https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions120.htm#SQLRF00685

NVL2 accepts 3 parameters. If the first parameter is NOT NULL, it will output parameter 2, otherwise it will output parameter 3.

e.g.

```sql
select
    nvl2(NULL, 'NON NULL val', 'NULL val') first_null,
    nvl2('Value', 'NON NULL val', 'NULL val') second_null
from dual
```
```
FIRST_NULL SECOND_NULL
---------- ------------
NULL val   NON NULL val
```

You may also be interested in the [NVL](NVL.md) function which has the reverse effect of getting the previous row.
