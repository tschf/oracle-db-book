# LTRIM

`ltrim` strips character(s) off the left hand side of a string. The first parameter is the string you'd like to trim from, and the next parameter is the character(s) you'd like to trim off.

The default character to be trimmed (if none is specified) is a single blank space. If more than one character is specified, it searches for any occurrence of that character up until a character not in the list - that is, it's not searching for the character(s) in sequence, but any of the specified characters.

```sql
select
    ltrim('  abracadabra') ltrim_1
  , ltrim('defabracadabra', 'dfe') ltrim_2  
from dual
```
Output:

```
LTRIM_1     LTRIM_2
----------- -----------
abracadabra abracadabra
```

See also [`rtrim`](rtrim.md) and [`trim`](trim.md).
