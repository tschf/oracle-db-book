# CURRENT_DATE

`current_date` is used to get the current date, obeying the sessions time zone. This differs from [`sysdate`](SYSDATE.md) which gets the date according the localisation of the database server.


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
