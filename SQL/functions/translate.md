#TRANSLATE

`translate` is a way to replace characters in one string with that of another. The first parameter is the string that you would like to search, the next (from) is the characters you would like to replace, and the third (to), what you would like them replaced with.

For each character in the second parameter, the character in the matching position is used to replace the formers value.

Some gotcha's worth noting:

* If a character in the from sequence appears twice, only the first replacement character would be used (in the relevant position)
* The replacement string must not be `NULL` (or any empty string) - for stripping characters out. A workaround here is to add a from and character replacement char in the first position and no other replacement characters.

```sql
select
    translate('967-123-5444', '9-', '9') strip_hyphen
  , translate('elephant', 'epet', 'carx') replace_epet
from dual
```

Output:
```
STRIP_HYPHEN REPLACE_EPET
------------ ------------
9671235444   clcahanx    
```
