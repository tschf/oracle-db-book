#CHR

`chr` returns character of the specified ascii code.

```sql
select
    chr(97) ascii_a
  , chr(98) ascii_b  
from dual
```

Output
```
ASCII_A ASCII_B
------- -------
a       b  
```

See [`ascii`](ascii.md) to go the other way.
