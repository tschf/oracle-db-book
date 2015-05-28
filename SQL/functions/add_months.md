#ADD_MONTHS

`add_months` adds the given number of months to the specified date, where the first parameter is the date you want to add the months to, and the second parameter is how many months to add.

If the date you are adding months to has a bigger date value than possible in the return month, the return value will be the maximum date for the return month.

Or if the date you are adding months to is the last day of the month, the return date will always return the last day of the month for the return month.

```sql
select
    add_months('31-JAN-2015', 1) date_1
  , add_months('28-FEB-2015', 1) date_2
from dual
```

Output:

```
DATE_1    DATE_2  
--------- ---------
28/FEB/15 31/MAR/15

```
