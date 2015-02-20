# LEAD

Oracle documentation: https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions087.htm#SQLRF00657

LEAST returns the smallest of a set of values. The return type is based on the first value passed in.

```sql
select least('5', 2, 4)
from dual
```

Would return 2 as a varchar2 data type.

Similarly, if you passed in a number first, it would return a number data type.

You may immediately think of the min function, however these are different in that least operates on a parameterised list of values - min operates on a column expression that evaluates a set of rows. [Read more here](MIN.md).
