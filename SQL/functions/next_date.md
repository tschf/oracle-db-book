#NEXT_DATE

`next_date` returns the next date, after the specified date, for the given day. The day parameter operates off the basis of day abbreviations. That is, if the first n chars are an abbreviation of a day, any following character will be ignored.

```sql
select
    sysdate
  , next_day(sysdate, 'MON') next_mon
  , next_day(sysdate, 'MONDzyx') next_mon2
  , next_day(sysdate, 'MONDAY') next_mon3
from dual
```
Output:
```
SYSDATE   NEXT_MON  NEXT_MON2 NEXT_MON3
--------- --------- --------- ---------
01/JUN/15 08/JUN/15 08/JUN/15 08/JUN/15
```
