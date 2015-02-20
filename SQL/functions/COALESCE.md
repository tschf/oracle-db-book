# COALESCE

Oracle documentation: https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions030.htm#SQLRF00617

COALESCE returns the first non NULL value in a set of values. It requires that all values passed in be the same data type.

e.g. the following won't implicitly convert varchar2's to NUMBERs

```sql
select coalesce(1, '6') output
from dual
```
```
ORA-00932: inconsistent datatypes: expected NUMBER got CHAR
```

This differs from nvl, which will do the implicit conversion:

```sql
select nvl(1, '6')
from dual
```
```
    OUTPUT
----------
         6
```
