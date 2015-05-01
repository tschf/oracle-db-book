# CARDINALITY

`cardinality` is used to get the number of rows in a nested table.

```sql
create type address_t is table of varchar2(50);
/

create table address_list(
    ID NUMBER primary key
  , address_list address_t
)
nested table address_list store as nested_address_list1;
/

insert into address_list values (1, address_t('15 Smith St', '16 Oxford Ave'));
/

commit;
/

select id, cardinality(address_list) num_adds
from address_list
```
Output:
```
        ID   NUM_ADDS
---------- ----------
         1          2

```

And using [`collect`](COLLECT.md):

```sql
drop table address_list;
/

create table address_list(
    ID NUMBER primary key
  , address varchar2(50)
);
/

insert into address_list values (1, '15 Smith St');
insert into address_list values (2, '16 Oxford Av');
/

commit;
/

select cardinality(cast(collect(address) as address_t)) num_adds
from address_list
```
Output:
```
  NUM_ADDS
----------
         2
```
