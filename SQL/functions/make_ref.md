# MAKE_REF

`make_ref` provides a way to fetch an object reference from an object view.

```sql
create type address_t as object (
    ID number,
    line1 varchar2(200),
    line2 varchar2(200),
    city varchar2(35),
    postcode varchar2(10)
);
/

create table address_list of address_t;
/

create view v_address_list of address_t
    with object identifier (id) as

    select id, line1, line2, city, postcode
    from address_list;
/



insert into address_list values (1, '26 Oxford St',NULL,'Sydney',2010);
insert into address_list values (2, '1600 Pennsylvania Avenue ',NULL,'Washington, DC',20500);
/

select make_ref(v_Address_list, id) add_ref
from v_Address_list;

```
Output:
```
ADD_REF
----------------
00004A038A00460FF410E3D79EC88CE050007F010004AC0000001426010001000100290000000000090600812A00078401FE0000000A02C1020000000000000000000000000000000000000000
00004A038A00460FF410E3D79EC88CE050007F010004AC0000001426010001000100290000000000090600812A00078401FE0000000A02C1030000000000000000000000000000000000000000


```
