# INITCAP

`initcap` is used to return a string in title case. That is, the first letter of each word will be in a capital, with every other letter being in lowercase.

```sql
with text as (
select 'a lower case string' str from dual union all
select 'AN UPPER CASE STRING' str from dual union all
select 'A mixEd CaSe STRing' str from dual union all
select '123456789!' str from dual
)
select str, initcap(str)
from text
```
Output:
```
STR                  INITCAP(STR)
-------------------- --------------------
a lower case string  A Lower Case String  
AN UPPER CASE STRING An Upper Case String
A mixEd CaSe STRing  A Mixed Case String  
123456789!           123456789!           

```
