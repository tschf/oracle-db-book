# COLLECT

COLLECT is used to return all the values into a nested table. To be useful, you need to have an custom data type to cast the data to, as demonstrated in the example below.

```sql
create table address(
    ID NUMBER PRIMARY KEY
  , ADDRESS VARCHAR2(200)
);
/

insert into address values (1, '15 Shoestring st');
insert into address values (2, '7 Pleasant av');
insert into address values (3, '1 Highway rd');
insert into address values (4, '9 Osprey pl');
insert into address values (5, '255 Jane ct');
insert into address values (6, '12 Woodville rd');

create type address_t is table of varchar2(200);
/
```

Then we can use collect:

```sql
select cast(collect(address) as address_t) addresses
from address
```

```
ADDRESSES
------------------------------
HR.ADDRESS_T('15 Shoestring st','7 Pleasant av','1 Highway rd','9 Osprey pl','255 Jane ct','12 Woodville rd')
```

If we a table with a column that is a nested table of the same type:

```sql

create table address2(
  ID NUMBER PRIMARY KEY
, address_list address_t
)
nested table address_list store as nested_address_list;
/
```

We could insert values using the collect function like so:

```sql
insert into address2 values
(
  1,
  (
    select cast(collect(address) as address_t)
    from address
  )
);
/
```

And you can loop over each address in a PL/SQL code block like so:

```sql
declare
    l_addresses address_t;
begin

    select cast(collect(address) as address_t)
    into l_addresses
    from address;

    for i in 1..l_addresses.COUNT
    LOOP

        dbms_output.put_line(l_addresses(i));

    END LOOP;
end;
```
