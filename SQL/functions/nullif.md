#NULLIF

`nullif` compares two values and if they are equal, will return `NULL`, otherwise the value of the first parameter will be returned.

```sql
select
    nullif('a', 'b') test1
  , nullif('a', 'a') test2
from
    dual
```

```
TEST1 TEST2
----- -----
a     (NULL)

```
