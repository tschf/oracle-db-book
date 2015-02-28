# LNNVL

`lnnvl` provides returns the inverse boolean value, whilst taking into account NULL values.


```sql
create table comparisons(
    ID NUMBER PRIMARY KEY
  , VAL NUMBER);
/

insert into comparisons values (1, NULL);
insert into comparisons values (2, 1);
insert into comparisons values (3, 2);
insert into comparisons values (4, 3);
insert into comparisons values (5, 4);
/

select
    id
  , val
  , case when val > 2 then 'Greater than 2' end compare1
  , case when lnnvl(val > 2) then 'Greater than 2' end compare2
from comparisons


```
Output:
```
        ID        VAL COMPARE1       COMPARE2
---------- ---------- -------------- --------------
         1                           Greater than 2
         2          1                Greater than 2
         3          2                Greater than 2
         4          3 Greater than 2
         5          4 Greater than 2
```
