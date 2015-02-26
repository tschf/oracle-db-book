# REF

Oracle documentation: https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions145.htm#SQLRF00694

`ref` is useful to get the object reference value for a row of an object table. As from the the [Oracle Database Object-Relational Developer's Guide](https://docs.oracle.com/cd/E11882_01/appdev.112/e11822/adobjdes.htm#ADOBJ00805), the returned value consists of (in order):

* 16 bytes for the system generated object ID of the object reference
* 16 bytes for the object ID of the table or view containing the data
* 10 bytes for the rowid hint

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

select ref(addl)
from address_list addl
```
Output:
```
REF(ADDL)
--------------------------------------------------------------------------------
00002802090FF2427B13E7FF3CE050007F010004AE0FF2427B13E6FF3CE050007F010004AE010051
B70000

00002802090FF2427B13E8FF3CE050007F010004AE0FF2427B13E6FF3CE050007F010004AE010051
B70001
```

We can use these references in a table:

```sql
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

select *
from address_list2
```
Output:
```
ID         ADDL_REF
---------- --------------------------------------------------------------------------
 1         00002202080FF410E3D788C88CE050007F010004AC0FF2427B13E6FF3CE050007F010004AE
 2         00002202080FF2427B13E7FF3CE050007F010004AE0FF2427B13E6FF3CE050007F010004AE

```

The reference is automatically de-referenced when referring to columns in the object. You must reference it using the table alias.

```sql
select id, addl.addl_Ref.line1
from address_list2 addl
```
Output:
```
ID ADDL_REF.LINE1
---------- --------------------------
 1 1600 Pennsylvania Avenue  
 2 26 Oxford St
```
