# Advanced Objects

The object model in the Oracle database is offered as an extension to the relational model - the basics of creating an Object in the Oracle Database were covered in the custom_types chapter. As a quick refresher, we could create an object to represent a person as

(types can only be used within the single database. see: https://docs.oracle.com/cd/E11882_01/appdev.112/e11822/adobjbas.htm#ADOBJ7083)

```sql
create or replace type person_typ is object (
    id NUMBER,
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
You can create a primary key on an object table by referencing an object's attribute.

```sql
create table objtable_person2 of person_typ (id primary key);
/
```
And, in the case of having an object column, you can add constraints as you would usually expect - using dot notation for the column name and object attribute.

```sql
create table table_person2(
    id number,
    person person_typ
);
/

alter table table_person2
add constraint "TABLE_PERSON2_UK1" UNIQUE (person.first_name, person.last_name);
/
```

Both object tables and objects as a column support regular insertion and querying by constructing the object type. When querying the object table can just reference individual fields as if they were regular columns.

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

From the context of PL/SQL, before an object is initialised, it is known to `atomically NULL`. That is the object as a whole is NULL. Each of it's attributes will also be NULL (and can be tested) before being initialised. In the reverse, if you initialise the object with all fields as NULL - memory has been allocated, so the object is no longer NULL.

```plsql
set serveroutput on

declare
    l_person person_typ;

    procedure test_null(p_person person_typ)
    as
    begin
        if p_person is null
        then
            dbms_output.put_line('NULL');
        else
            dbms_output.put_line('Not NULL');
        end if;
    end;    
begin

    test_null(l_person);--NULL

    l_person := person_typ(NULL,NULL,NULL);
    test_null(l_person);--Not NULL


end;
/
```

So far, the examples have only been using a type specification, where to initialise the the object you need to have a value for each attribute. e.g. `l_person := person_typ('John', 'Smith', to_date('01-01-2001', 'DD-MM-YYY'))`. We can now extend on this, to introduce constructors and methods against the object.

They can be classed into:

* Member functions/procedures
* Static functions/procedures
* Consutors

## Member functions/procudures

These have access to the instance attributes. They can be accessed through the `SELF` field. In the specification, you declare it with `member function func_name return type`. SELF is always available without the need to declare it - although, you can optionally pass it in, in the parameter list `SELF in out nocopy person_typ2`.

```plsql
create or replace type person_typ2 is object (
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE,
    member function get_name return varchar2,
    member function get_name2(SELF in out nocopy person_typ2) return varchar2
);
/

create or replace type body person_typ2 as

    member function get_name return varchar2
    as
    begin
        return self.first_name || ' ' || last_name;--can include or exclude `SELF`
    end get_name;

    member function get_name2(SELF in out nocopy person_typ2) return varchar2
    as
    begin
        return self.first_name || ' ' || last_name;
    end get_name2;
end;    

/

declare
    l_person person_typ2;
begin


    l_person := person_typ2('John', 'Smith', NULL);

    dbms_output.put_line(l_person.get_name());--John Smith
    dbms_output.put_line(l_person.get_name2());--John Smith

end;
/
```

Whilst you don't need to explicitly declare object attributes with dot notation against `SELF`, I believe it gives the program that extra clarity, so would suggest using it in your methods. Not only that, but the default behaviour is not with the compiler hint `NOCOPY`, so if your objects hold a large amount of data, you may notice a performance hit.

## Static methods

If you don't require any instance values, you can use a static method. Which is done with the syntax `static function` or `static procedure`. You then use the type name rather than a variable of the object.

```plsql
create or replace type person_typ2 is object (
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE,
    static function get_count return NUMBER
);
/

create or replace type body person_typ2 as

    static function get_count return NUMBER
    as
    begin
        return 1;--todo:implement count
    end get_count;
end;    
/

set serveroutput on

declare
    l_person person_typ2;
begin

    l_person := person_typ2('John', 'Smith', NULL);
    dbms_output.put_line(person_typ2.get_count());--1

end;
/
```

## Constructors

By default, an object has an implicit constructor, which expects a value for each defined field. The previous examples have demonstrated this with `person_typ(NULL,NULL,NULL)` - setting a NULL value for each field. Suppose however, that you want to support instantiation of your object without passing in the hire date (e.g. just passing in the first name and last name).

The function is declared with the clause `constructor function type_name(param_list) return self as result`. Then, at the end of the function, you issue a `return` statement. When instantiating the object, you can opt to use the `new` keyword, though this is not necessary.

```plsql
create or replace type person_typ2 is object (
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE,
    constructor function person_typ2(p_first_name in varchar2, p_last_name in varchar2) return self as result
);
/

create or replace type body person_typ2 as


    constructor function person_typ2(p_first_name in varchar2, p_last_name in varchar2) return self as result
    as
    begin
        self.first_name := p_first_name;
        self.last_name := p_last_name;
        return;
    end;
end;    
/

declare
    l_person person_typ2;
begin

    l_person := person_typ2('John', 'Smith');
    l_person := new person_typ2('John', 'Smith');
    --equivalent

end;
/
```
