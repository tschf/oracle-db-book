#ASCII

`ascii` returns the ascii code of the first character of the first passed in string.

```sql
select
    ascii('a') ascii_a
  , ascii('b') ascii_b  
  , ascii('bat') ascii_b2
from dual
```

Output
```
ASCII_A    ASCII_B   ASCII_B2
---------- ---------- ----------
     97         98         98
```

See [`chr`](chr.md) to go the other way.
