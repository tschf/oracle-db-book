# TO_YMINTERVAL

`to_yminterval` returns an `interval year to month` data type. It accepts one parameter - a string with the interval information. It supports the string in two different formats.

The first is the format as described on the [intervals](../concepts/INTERVALS.md) concept page which can include just the year value, or the year and month value separated by a hyphen.

```sql
select
    interval '02-01' year to month int_literal,
    to_yminterval('02-01') int_fnc
from dual
```
Output:
```
INT_LITERAL INT_FNC
----------- -------
+02-01      +02-01  

```

The other format is as per [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) standard. This standard describes the format as starting with the letter P (period) and being able to include each time component followed my the relevant modifier. If the time information is included, it beings with the letter T (time):

**P**
* Y - year
* M - month
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
Output:
```
INT_LITERAL INT_FNC INT_FNC2
----------- ------- --------
+02-01      +02-01  +02-01
```
