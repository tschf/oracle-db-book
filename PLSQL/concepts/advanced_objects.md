# Advanced Objects

The object model in the Oracle database is offered as an extension to the relational model - the basics of creating an Object in the Oracle Database were covered in the custom_types chapter. As a quick refresher, we could create an object to represent a person as

```plsql
create or replace type person_typ is object (
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE
);
/
```

Objects can be stored either as a column among built-in datatypes; or in an object table, which means each row represent the object in its entirety.

e.g.

```sql
create table objtable_person of person_typ;
/
--
create table table_person(
    id number,
    person person_typ
);
/
```
Both forms support regular insertion and querying by contstructing the object type. When querying the object table can just reference individual fields as if they were regular columns.

```sql
--object table
insert into objtable_person values (person_typ('John', 'Smith', sysdate));
/

select e.first_name, e.last_name, e.hire_date
from objtable_person e;
/

--object column
insert into table_person values (1, person_typ('John', 'Smith', sysdate));
/

select *
from table_person;
/
```

Observing the object table version, you may think how can we relate object tables to one another. Object rows have a `ref` which points to the row. We can declare a `ref` pointer to a particular object by using that keyword before the particular object type; and then use the `ref` function to retrieve the appropriate ref. When querying the table, to access the referenced column values, we would need to `deref`erence the object, to be able to access the fields.

```sql
create table table_pet(
    pet_name varchar2(50),
    pet_owner ref person_typ
);
/

insert into table_pet

select 'Spot', ref(p)
from objtable_person p
where first_name = 'John'
and last_name = 'Smith'
;
/

select
    tp.pet_name,
    deref(tp.pet_owner).first_name owner_first,
    tp.pet_owner.last_name owner_last--implicit dereferencing
from table_pet tp;
/
```

//todo: mention about `scope is`

One downside here, is that there is nothing to stop the referenced object being removed. When this happens, the reference is known to be `dangling`. Your query can ensure the reference is not dangling with the `is not dangling` clause (or `is dangling` for the inverse check).

```sql
select
    pet_name,
    deref(pet_owner).first_name owner_first,
    deref(pet_owner).last_name owner_last
from table_pet
where pet_owner is not dangling;
/
```

But, the better approach would be to enforce some referential integrity to prevent references from dangling in the first place.


What this doesn't include is any constructors or functions. These would typically be introduced in the object body (with the exposed functions also specified in the type specification).
