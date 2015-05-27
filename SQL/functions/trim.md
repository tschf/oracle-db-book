# TRIM

`trim` strips a character from either end of a string. If the input is just the string to trim, it will strip blank spaces from both ends of the string. You can however specify a the character you'd like to trim as well as if its from the `leading` or `trailing` point of the string.

```sql
select
    trim(' abracadabra ') trim_1
  , trim('0' from '0abracadabra0') trim_2
  , trim(both '0' from '0abracadabra0') trim_3
  , trim(leading '0' from '0abracadabra0') trim_3
  , trim(trailing '0' from '0abracadabra0') trim_3
from dual
```
Output:

```
TRIM_1      TRIM_2      TRIM_3      TRIM_4       TRIM_5
----------- ----------- ----------- ------------ ------------
abracadabra abracadabra abracadabra abracadabra0 0abracadabra
```
See also [`ltrim`](ltrim.md) and [`rtrim`](rtrim.md).
