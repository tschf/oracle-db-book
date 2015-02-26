# LOWER

`lower` converts every single character in a string to be lowercase.

```sql
with text as (
select 'a lower case string' str from dual union all
select 'AN UPPER CASE STRING' str from dual union all
select 'A mixEd CaSe STRing' str from dual union all
select '123456789!' str from dual
)
select str, lower(str)
from text
```
Output:
```
STR                  LOWER(STR)
-------------------- --------------------
a lower case string  a lower case string  
AN UPPER CASE STRING an upper case string
A mixEd CaSe STRing  a mixed case string  
123456789!           123456789!           
```
