# NUMTOYMINTERVAL

Used to convert a number into an interval of `year` or `month`.

The first parameter is the number of units, and the second is the units you would like to return. Acceptable values for the units are `year` or `month` case-insensitive.

```sql
select
    numtoyminterval(1, 'year') yr1,
    numtoyminterval(1, 'month') mn1,
    numtoyminterval(1, 'year') + numtoyminterval(1, 'month') yr1_mn1
from dual
```
Output:
```
YR1    MN1    YR1_MN1
-----  -----  -------
+01-00 +00-01 +01-01
```
