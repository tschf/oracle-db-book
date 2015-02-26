# REFTOHEX

Similar to `ref`, this function gets the hex ref of an object.


```sql
create type address_t as object (
    line1 varchar2(200),
    line2 varchar2(200),
    city varchar2(35),
    postcode varchar2(10)
);
/

create table address_list of address_t;
/

insert into address_list values ('26 Oxford St',NULL,'Sydney',2010);
insert into address_list values ('1600 Pennsylvania Avenue ',NULL,'Washington, DC',20500);
/

create table address_list2(
    ID NUMBER PRIMARY KEY
  , ADDL_REF REF address_t scope is address_list
);
/

insert into address_list2
select
    row_number() over (order by addl.line1) id
  , ref(addl) addl_ref
from address_list addl;
/

select addl_ref
from address_list2 addl
```
Output:
```
ADDL_REF
--------------
00002202080FF410E3D7B1C88CE050007F010004AC0FF410E3D7AFC88CE050007F010004AC
00002202080FF410E3D7B2C88CE050007F010004AC0FF410E3D7AFC88CE050007F010004AC


```
Using `reftohex` on the ref

```sql
select reftohex(addl_ref)
from address_list2 addl
```
Output:
```
REFTOHEX(ADDL_REF)
--------------------------------------------------------------------------
00002202080FF410E3D7B1C88CE050007F010004AC0FF410E3D7AFC88CE050007F010004AC
00002202080FF410E3D7B2C88CE050007F010004AC0FF410E3D7AFC88CE050007F010004AC
```
