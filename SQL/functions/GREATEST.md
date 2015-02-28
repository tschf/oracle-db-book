# GREATEST

GREATEST returns the largest of a set of values. The return type is based on the first value passed in.

```sql
select greatest('5', 2, 4)
from dual
```

Would return 5 as a varchar2 data type.

Similarly, if you passed in a number first, it would return a number data type.

You may immediately think of the `max` function, however these are different in that `greatest` operates on a parameterised list of values. `max` operates on a column expression that evaluates a set of rows. [Read more here](MAX.md).
