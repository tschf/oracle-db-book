# TO_DSINTERVAL

`to_dsinterval` returns an `interval day to second` data type. It accepts one parameter - a string with the interval information. It supports the string in two different formats.

The first is the format as described on the [intervals](../concepts/INTERVALS.md) concept page which should include both the days and time portion.

```sql
select
    interval '01 1:1:1.111' day to second int_literal,
    to_dsinterval('01 1:1:1.111') int_fnc
from dual
```
Output:
```
INT_LITERAL INT_FNC
----------- -----------
1 1:1:1.111 1 1:1:1.111

```

The other format is as per [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) standard. This standard describes the format as starting with the letter P (period) and being able to include each time component followed my the relevant modifier. If the time information is included, it beings with the letter T (time):

**P**
* D - day

**T**
* H - hour
* M - minute
* S - second

Whilst the input can include timing information, because this is for a year to month interval, any timing information will be lost in the interval.

```sql
select
    interval '02-01' year to month int_literal,
    to_yminterval('P02Y01M') int_fnc,
    to_yminterval('P02Y01MT01H01M01.01S') int_fnc2
from dual
```
