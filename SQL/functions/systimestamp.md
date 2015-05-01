# SYSTIMESTAMP

`systimestamp` returns the current timestamp according to the localisation of the database server. This differs from [`current_timestamp`](CURRENT_TIMESTAMP.md) which returns the timestamp according to the localisation of the database server.

```sql
alter session
set time_zone = 'US/Central';
/

select
    sessiontimezone
  , current_timestamp time_us_central
  , systimestamp time_sydney
from dual
```
Output:
```
SESSIONTIMEZONE  TIME_US_CENTRAL                            TIME_SYDNEY
---------------- ------------------------------------------ ----------------------------------------
US/Central       21/FEB/15 03:36:59.177109000 PM US/CENTRAL 22/FEB/15 08:36:59.177102000 AM +11:00
```
