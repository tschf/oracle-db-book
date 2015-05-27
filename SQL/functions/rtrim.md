# RTRIM

`rtrim` strips character(s) off the right hand side of a string. The first parameter is the string you'd like to trim from, and the next parameter is the character(s) you'd like to trim off.

The default character to be trimmed (if none is specified) is a single blank space. If more than one character is specified, it searches for any occurrence of that character from the end of the string up until not of the specified characters are found.

```sql
select
    rtrim('abracadabra   ') rtrim_1
  , rtrim('abracadabradef', 'dfe') rtrim_2  
from dual
```
Output:

```
RTRIM_1     RTRIM_2
----------- -----------
abracadabra abracadabra
```
See also [`ltrim`](ltrim.md) and [`trim`](trim.md).
