# SYSDATE

`sysdate` returns the current date according to the localisation of the database server. This differs from [`current_date`](CURRENT_DATE.md) which returns the date, obeying the sessions time zone.

```sql
alter session
set time_zone = 'US/Central';
/

select
    sessiontimezone
  , to_char(current_date, 'DD-MON-YYYY HH24:MI.SS') date_us_central
  , to_char(sysdate, 'DD-MON-YYYY HH24:MI.SS') date_sydney
from dual
```
Output:
```
SESSIONTIMEZONE    DATE_US_CENTRAL               DATE_SYDNEY
------------------ ----------------------------- -----------------------------
US/Central         21-FEB-2015 15:30.53          22-FEB-2015 08:30.53

```
