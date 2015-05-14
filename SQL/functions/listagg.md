#LISTAGG

`listagg` is an aggregate function that allows you to collate the results from multiple rows into one, concatenating with the specified parameter.

```sql
select listagg(region_name, ', ') within group (order by region_id)
from regions
```

Outut:
```
REGION_LIST
--------------------------------------------------------------------------------
Europe, Americas, Asia, Middle East and Africa
```

It can also be used as an analytic function:

```sql
select region_id, listagg(region_name, ', ') within group (order by region_id) over () region_list
from regions
```

Output:

```
REGION_ID REGION_LIST
--------- --------------------------------
        1 Europe, Americas, Asia, Middle East and Africa
        2 Europe, Americas, Asia, Middle East and Africa
        3 Europe, Americas, Asia, Middle East and Africa
        4 Europe, Americas, Asia, Middle East and Africa
```
