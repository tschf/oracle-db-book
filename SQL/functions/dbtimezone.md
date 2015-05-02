#DBTIMEZONE

`dbtimezone` returns the currently set timezone of the database.

```sql
select dbtimezone
from dual
```

Output:
```
DBTIMEZONE
----------
+00:00
```

The timezone can be changed as described in the [Oracle Database Globalization Support guide](http://docs.oracle.com/cd/E11882_01/server.112/e10729/ch4datetime.htm#NLSPG262).
