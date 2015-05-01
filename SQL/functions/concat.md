#CONCAT

`concat` is used to concatenate two values together, which accepts character datatypes.

```sql
select concat('abc','def') new_field
from dual;
/
```
Output:
```

NEW_FIELD
---------
abcdef
```

More useful would be the concatenation operator `||`. See here: http://docs.oracle.com/cd/E11882_01/server.112/e41084/operators003.htm#SQLRF51158
