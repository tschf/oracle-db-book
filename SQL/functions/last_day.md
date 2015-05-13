#LAST_DAY

`last_day` accepts either a `date` or `timestamp` variable, and returns the last day of the month for the date of the passed in value. No matter what data type you pass in, the return type will be a `date`.

```sql
select
    sysdate date_current  
  , last_day(sysdate) date_last
  , systimestamp ts_current
  , last_day(systimestamp) ts_last
from dual
```

Output:
```
DATE_CURRENT DATE_LAST TS_CURRENT                               TS_LAST
------------ --------- ---------------------------------------- ---------
13/MAY/15    31/MAY/15 13/MAY/15 07:58:06.601566000 AM -04:00   31/MAY/15
```
