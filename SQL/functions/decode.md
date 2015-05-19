#DECODE

`decode` is quite similar to a `case` statement. The first parameter is the value you are testing the value of, then the following parameters go in sets of two, with the first parameter being the value to test for, and the return value if that value is matched. Then finally one more parameter which represents what should be returned when no value was found.

```sql
select
    region_id
  , region_name
  , decode(region_name, 'Europe', 'euro', 'Americas', 'usd', 'unknown') currency
from regions
```

Output:
```
REGION_ID  REGION_NAME               CURRENCY
---------- ------------------------- -------
        1 Europe                    euro
        2 Americas                  usd
        3 Asia                      unknown
        4 Middle East and Africa    unknown
```
