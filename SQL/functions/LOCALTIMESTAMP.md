# LOCALTIMESTAMP

Oracle documentation: https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions092.htm#SQLRF00660

`localtimestamp` is used to get the current timestamp, ignoring the sessions time zone. This differs from `systimestamp` which gets the date according the localisation of the database server. The value returned does not include timezone information. If you want time zone information returned, you should use [CURRENT_TIMESTAMP](CURRENT_TIMESTAMP.md)

```sql
alter session
set time_zone = 'US/Central';
/

select
    sessiontimezone
  , localtimestamp time_us_central
  , systimestamp time_sydney
from dual
```
Output:
```
SESSIONTIMEZONE  TIME_US_CENTRAL                   TIME_SYDNEY
---------------- --------------------------------- ----------------------------------------
US/Central       21/FEB/15 03:47:09.593712000 PM   22/FEB/15 08:47:09.593706000 AM +11:00
```
