# Multisets

Multisets, are really just nested tables. The [`cast`](../functions/CAST.md) function is particularly useful here in converting your data into a nested table. Two common functions you would use in combination with [`cast`](../functions/CAST.md) are [`collect`](../functions/COLLECT.md) - which can convert into a nest table of a single data type - and `multiset` - which accepts a subquery. See the related documentation pages for specific examples.

The following set operations that are supported on multiple queries, are also supported on nested tables.

* UNION - distinct rows
* UNION ALL - all rows  
* INTERSECT - common rows  
* MINUS - remove rows from the first dataset that are in the second dataset  

With some subtle differences. The `union all` is not available to use, with `union` having the same behaviour of `union all` when working with queries. Instead of `minus` you would use `except`. You also use the `multiset` keyword before the set operation.

```sql
create type t_address is object (
    street_address varchar2(40),
    post_code varchar2(12)
);
/

create type t_address_list is table of t_address;
/

alter table employees_demo
add (
    add_list t_address_list
)
NESTED TABLE add_list store as add_list_store
;
/

alter table employees_demo
add (
    add_list2 t_address_list
)
NESTED TABLE add_list2 store as add_list2_store
;
/

update employees_demo
set add_list = cast(multiset(
select street_address, postal_code
from locations
where location_id in (1000, 1100)) as t_address_list)
where employee_id = 102;
/

update employees_demo
set add_list2 = cast(multiset(
select street_address, postal_code
from locations
where location_id in (1100, 1200)) as t_address_list)
where employee_id = 102;
/

```

## Union

```sql
select add_list multiset union add_list2
from employees_demo
where employee_id = 102
```
Output:
```
UNIONED_MULTISET
--------------------
HR.T_ADDRESS_LIST(
    HR.T_ADDRESS('1297 Via Cola di Rie','00989'),
    HR.T_ADDRESS('93091 Calle della Testa','10934'),
    HR.T_ADDRESS('93091 Calle della Testa','10934'),
    HR.T_ADDRESS('2017 Shinjuku-ku','1689')
)
```

## Intersect

```sql
select add_list multiset intersect add_list2 intersect_multiset
from employees_demo
where employee_id = 102
```
Output:
```
INTERSECT_MULTISET
--------------------
HR.T_ADDRESS_LIST(
    HR.T_ADDRESS('93091 Calle della Testa','10934')
)  
```

## Except

```sql
select add_list multiset except add_list2 except_multiset
from employees_demo
where employee_id = 102
```
Output:
```
UNIONED_MULTISET
--------------------
HR.T_ADDRESS_LIST(
    HR.T_ADDRESS('1297 Via Cola di Rie','00989')
)
```
