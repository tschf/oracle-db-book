# NUMTODSINTERVAL

Used to convert a number into an interval of `day`, `hour`, `minute` or `second`.

The first parameter is the number of units, and the second is the units you would like to return. Acceptable values for the units are `day`, `hour`, `minute` or `second` case-insensitive.

```sql
select
    numtodsinterval(1, 'day') dy1,
    numtodsinterval(1, 'hour') hr1,
    numtodsinterval(1, 'minute') mn1,
    numtodsinterval(1, 'second') sc1,
    numtodsinterval(1, 'day') +
    numtodsinterval(1, 'hour') +
    numtodsinterval(1, 'minute') +
    numtodsinterval(1, 'second') dyhrmnsc1
from dual
```
Output:
```
DY1         HR1         MN1         SC1         DYHRMNSC1  
----------- ----------- ----------- ----------- -----------
1 0:0:0.0   0 1:0:0.0   0 0:1:0.0   0 0:0:1.0   1 1:1:1.0
```
