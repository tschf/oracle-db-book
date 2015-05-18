#FLOOR

`floor` removes the decimal portion of a number, to give the closest whole number. That is 15.1, 15.5, 15.99, floored, would all return the whole number of 15.

```sql
select
    floor(15.1) floor_1
  , floor(15.5) floor_2
  , floor(15.99) floor_3
from
    dual
```

Output:
```
FLOOR_1    FLOOR_2    FLOOR_3
---------- ---------- ----------
     15         15         15
```
