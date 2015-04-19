#CONVERT

`convert` is used to change the character set of a particular string. The first parameter is the string you are wishing to convert followed by the source character set and then the destination character set.

Current versions of an Oracle database support two character sets:

1. Database character set
* National character set

Since any other character set is not supported, this function should only be used when a string value is not in one of the two aforementioned character sets.

A full list of valid character sets can be obtained by querying the view `V$NLS_VALID_VALUES`.

```sql
select convert('Résumé', 'UTF8','WE8ISO8859P1')
from dual
```

Output:
```
RESUME_1
-------------
R Ã© s u m Ã©
```
