#CEIL

If a number has a greater decimal value then n.0, `ceil` will return the next whole integer

1 returns 1  
1.00001 returns 2

```sql
select ceil(1) one_1, ceil(1.00001) one_2
from dual
```

Output:

```
ONE_1      ONE_2
---------- ----------
    1          2
```
