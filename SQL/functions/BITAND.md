# BITAND

`bitand` ANDs the binary values of two inputs. It expects the input to be in base 10, then ANDs the two numbers together.

If we wanted to AND 10 and 15 together we would be ANDing the values:

1 0 1 0 and  
1 1 1 1

Giving us:

1 0 1 0 -- 10

```sql
select
    bitand(10,15) ten_fifteen
from dual
```
Output:
```
TEN_FIFTEEN
-----------
         10 
```
