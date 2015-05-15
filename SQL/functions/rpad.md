#RPAD

`rpad` adds padding to the right of a string. The first parameter is the string you would like to add padding to, the second is the minimum length you would like your return string to be, and the final is the character(s) you would like to have your string padded with (to the right).

If the length of the passed in string is already longer than the length specified, no change will happen.

If you specify more than one padding character, they will be inserted in sequence until the desired length of the string is reached.

```sql
select
    rpad('test', 4, '*#') pad_4_4
  , rpad('test', 5, '*#') pad_4_5
  , rpad('test', 6, '*#') pad_4_6
  , rpad('test', 7, '*#') pad_4_7
from dual
```

Output:
```
PAD_4_TO_4 PAD_4_TO_5 PAD_4_TO_6 PAD_4_7
---------- ---------- ---------- -------
test       test*      test*#     test*#*
```

See also [`lpad`](lpad.md).
