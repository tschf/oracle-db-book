# NVL2

The idea of NVL is to allow the user to return a specific value in the case the first parameter is NULL. It's similar to [COALESCE](COALESCE.md) however NVL only accepts two parameters, and performs implicit conversion where possible.

e.g.

```sql
select nvl(1, '6') output
from dual
```
```
    OUTPUT
----------
         1
```

You may also be interested in the [NVL2](NVL2.md) function(s).
