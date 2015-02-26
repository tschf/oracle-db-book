# UPPER

`upper` converts every single character in a string to be uppercase.

```sql
with text as (
select 'a lower case string' str from dual union all
select 'AN UPPER CASE STRING' str from dual union all
select 'A mixEd CaSe STRing' str from dual union all
select '123456789!' str from dual
)
select str, upper(str)
from text
```
Output:
```
STR                  UPPER(STR)
-------------------- --------------------
a lower case string  A LOWER CASE STRING  
AN UPPER CASE STRING AN UPPER CASE STRING
A mixEd CaSe STRing  A MIXED CASE STRING  
123456789!           123456789!

```
