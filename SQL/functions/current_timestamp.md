# CURRENT_TIMESTAMP

`current_timestamp` is used to get the current timestamp, obeying the sessions time zone. This differs from `systimestamp` which gets the date according the localisation of the database server. The value returned includes timezone information. If you don't want time zone information returned, you should use [LOCALTIMESTAMP](localtimestamp.md)


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
