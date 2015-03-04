# INTERVALS

Intervals are a convenient way to refer to units of time. They allow you to perform date arithmetic and are in terms of years, months, days, hours, minutes or seconds. The two datatypes that correspond to intervals are:

1. INTERVAL YEAR TO MONTH
* INTERVAL DAY TO SECOND

If the year unit is 3 or more digits, you **must** specify the precision as the default precision is 2.

You can initialise an interval by using the `interval` keyword, followed by a numeric value, and finally the unit the interval will be in - year, month, day, hour, minute or second.

As a basic example, each different unit can be defined like so:

```sql
interval '1' second
interval '1' minute
interval '1' hour
interval '1' day
interval '1' month
interval '1' year
```

With that we can perform date arithmetic on `date` values:

```sql
with a_date as (
    select to_date('01/03/2015 10:30:00', 'dd/mm/yyyy hh24:mi:ss') first_march
    from dual
)

select
    to_char(first_march, 'dd/mm/yyyy hh24:mi:ss') first_march
  , to_char(first_march - interval '1' second, 'dd/mm/yyyy hh24:mi:ss') less_sec
  , to_char(first_march - interval '1' minute, 'dd/mm/yyyy hh24:mi:ss') less_min
  , to_char(first_march - interval '1' hour, 'dd/mm/yyyy hh24:mi:ss') less_hour
  , to_char(first_march - interval '1' day, 'dd/mm/yyyy hh24:mi:ss') less_day
  , to_char(first_march - interval '1' month, 'dd/mm/yyyy hh24:mi:ss') less_month
  , to_char(first_march - interval '1' year, 'dd/mm/yyyy hh24:mi:ss') less_year
from a_Date
```

As well as timestamps:

```sql
with a_time as (
    select TO_TIMESTAMP ('01/03/2015 10:30:00.000000', 'dd/mm/yyyy HH24:MI:SS.FF') first_march
    from dual
)

select
    to_char(first_march, 'dd/mm/yyyy hh24:mi:ss') first_march
  , to_char(first_march - interval '1' second, 'dd/mm/yyyy hh24:mi:ss') less_sec
  , to_char(first_march - interval '1' minute, 'dd/mm/yyyy hh24:mi:ss') less_min
  , to_char(first_march - interval '1' hour, 'dd/mm/yyyy hh24:mi:ss') less_hour
  , to_char(first_march - interval '1' day, 'dd/mm/yyyy hh24:mi:ss') less_day
  , to_char(first_march - interval '1' month, 'dd/mm/yyyy hh24:mi:ss') less_month
  , to_char(first_march - interval '1' year, 'dd/mm/yyyy hh24:mi:ss') less_year
from a_time
```

Output:
```
FIRST_MARCH         LESS_SEC            LESS_MIN            LESS_HOUR           LESS_DAY            LESS_MONTH          LESS_YEAR
------------------- ------------------- ------------------- ------------------- ------------------- ------------------- -------------------
01/03/2015 10:30:00 01/03/2015 10:29:59 01/03/2015 10:29:00 01/03/2015 09:30:00 28/02/2015 10:30:00 01/02/2015 10:30:00 01/03/2014 10:30:00
```

In PL/SQL, you can't pass in a string variable to the number of units, but you can perform arithmetic on intervals.

```sql
declare
    l_time varchar2(3);
    l_new_date DATE;
begin
    l_time_unit := '2';

    l_new_date := sysdate + interval l_time_unit day;
    dbms_output.put_line(l_new_date);
end;

--PLS-00103: Encountered the symbol "L_TIME_UNIT" when expecting one of the following:
```

The above results in the mentioned error, so the better technique is to use `'1'` as the time unit for the interval, and have a multiplication factor variable, like so:

```sql
declare
    l_time_unit NUMBER;
    l_new_date DATE;
begin
    l_time_unit := 2;

    l_new_date := sysdate + (l_time_unit * interval '1' day);
    dbms_output.put_line(l_new_date);

end;
---06/MAR/15
```


In addition to the above examples, you can also have literals with a trailing unit of time, with support for two kinds:

* Year to month; and
* Day to second

You can have a year and month in a single interval literal by separating the number of months from the number of years with a hyphen (`'5-2'`) and adding the keywords `to month` after the `year` keyword.

The day to second interval supports going from day to hours, minutes or seconds. In this case you separate the hours/minutues/seconds expression from the day expression by a space, then specify the highest time unit to the lowest time unit. You can do:

* Day to hour
* Day to minute
* Day to second
* Hour to minute
* Hour to second
* Minute to second

```
interval '5-2' year to month --+05-02

interval '1 1' day to hour --+01 01:00:00.000000
interval '1 0:1' day to minute --+01 00:01:00.000000
interval '1 0:0:1.556' day to second(2) --+01 00:00:01.560000
interval '1:1' hour to minute --+00 01:01:00.000000
interval '1:0:1' hour to second --+00 01:00:01.000000
interval '1:1' minute to second --+00 00:01:01.000000
```

When performing arithmetic on date values, you can also return the year to month or day to second interval by wrapping the arithmetic in parenthesis, and passing the `year to month` or `day to second` literal.

```sql
select
  (to_date('03/03/2001', 'dd/mm/yyyy') - to_date('01/01/2001', 'dd/mm/yyyy')) year to month ym
, (to_date('03/03/2001', 'dd/mm/yyyy') - to_date('01/01/2001', 'dd/mm/yyyy')) day to second ms
from dual
```
Output:

```
YM      MS
------- -----------
+00-02  61 0:0:0.0
```

Aside from using the literals, there are also some functions that may be useful in getting an interval values:

* NUMTOYMINTERVAL
* NUMTODSINTERVAL
* TO_DSINTERVAL
* TO_YMINTERVAL
