#ASCIISTR

`asciistr` returns the ascii version of the passed in string. Non ascii characters are returned in UTF-16 backslash format. The character `é` for  instance would be returned as `\00E9`.

```sql
select asciistr('résumé') ascii_resume
from dual
```

Output:
```
ASCII_RESUME
--------------
r\00E9sum\00E9
```
