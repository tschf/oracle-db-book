# CAST

`cast` can be used to convert a value into a specific data type. You can for instance use `cast` instead of `to_number` when converting a string to a number.

```sql
create or replace view v_number_sample
as
select to_number('1') ver1, cast('1' as NUMBER) ver2
from dual;

desc v_number_sample;
```

Output:
```
Name Null Type
---- ---- ------
VER1      NUMBER
VER2      NUMBER
```

Or you may need to specify the the data precision, that is where cast can be particularly useful.


```sql
create or replace view v_char_sample
as
select '10 characters' ver1, cast('10 characters' as varchar2(200)) ver2
from dual;

desc v_char_sample;
```

Output:
```

Name Null Type
---- ---- -------------
VER1      CHAR(13)
VER2      VARCHAR2(200)
```

`cast` can also be used with the `multiset` function, to return a subquery as a nested table type.

```sql
create type locations_ot is object (
    location_id number(4,0),
    street_address varchar2(40),
    postal_code varchar2(12),
    city varchar2(30),
    state_province varchar2(25),
    country_id char(2)
);
/

create type locations_ntt is table of locations_ot;
/

select
    cast(
        multiset(
            select location_id, street_address, postal_code, city, state_province, country_id
            from locations
            where location_id in (1000,1100))
    as locations_ntt) locs
from dual

```

Output:

```
LOCS
---------
HR.LOCATIONS_NTT(HR.LOCATIONS_OT(1000,'1297 Via Cola di Rie','00989','Roma',NULL,'IT'),HR.LOCATIONS_OT(1100,'93091 Calle della Testa','10934','Venice',NULL,'IT'))  
```
