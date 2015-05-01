# DEREF

`deref` returns the object that is being referenced. Using the same data created in [`REF`](REF.md).

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

select id, deref(addl_ref)
from address_list2;
/
```
Output:
```
ID         DEREF(ADDL_REF)(LINE1, LINE2, CITY, POSTCODE)
---------- -----------------------------------------------------------------
 1         ADDRESS_T('1600 Pennsylvania Avenue ', NULL, 'Washington, DC', '20500')
 2         ADDRESS_T('26 Oxford St', NULL, 'Sydney', '2010')


```
