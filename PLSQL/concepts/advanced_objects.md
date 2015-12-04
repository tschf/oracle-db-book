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

Each object table has an object-identifier (OID). This can either be a system generated one (by default), or one that you specify as pointing to the tables primary key. In deciding if to use the system generated OID vs a primary key OID, the documentation states:

> Primary-key based identifiers make it faster and easier to load data into an object table. By contrast, system-generated object identifiers need to be remapped using some user-specified keys, especially when references to them are also stored. If you use system-generated OIDs for an object table, Oracle maintains an index on the column that stores these OIDs. A system-generated OID requires extra storage space for this index and an extra 16 bytes of storage for each row object.

> However, if each primary key value requires more than 16 bytes of storage and you have a large number of REFs, using the primary key might require more space than system-generated OIDs because each REF is the size of the primary key.

```sql
create table objtable_person3 of person_typ (id primary key)
object identifier is primary key;
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

Both object tables and objects as a column support regular insertion and querying by constructing the object type. When querying, the object table can just reference individual fields as if they were regular columns.

```sql
--object table
insert into objtable_person values (person_typ(1, 'John', 'Smith', sysdate));
/

select e.first_name, e.last_name, e.hire_date
from objtable_person e;
/

--object column
insert into table_person values (1, person_typ(1, 'John', 'Smith', sysdate));
/

select e.person.first_name, e.person.last_name, e.person.hire_date
from table_person e;--table must be aliased to refer to columns in the object type
/
```

Observing the object table version, you may think, how can we relate object tables to one another. Object rows have a `ref` which points to the row. We can declare a `ref` pointer to a particular object by using that keyword before the particular object type; and then use the `ref` function to retrieve the appropriate ref. When querying the table, to access the referenced column values, we would need to `deref`erence the object, to be able to access the fields.

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

You can optionally define a scope of your object references when declaring them. Scoped references don't require as much storage space, resulting in more efficient queries and as an added bonus, it becomes quite clear which table your object reference is relating to. The scope is set up with the `scope is` clause immediately after the object type, in the declaration.

```sql
create table table_pet2(
    pet_name varchar2(50),
    pet_owner ref person_typ scope is objtable_person
);
/

insert into table_pet2

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
from table_pet2 tp;
/
```

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

But, the better approach would be to enforce some referential integrity to prevent references from dangling in the first place. There is nothing really new here, foreign keys can be added using the alter table statement as you would expect with any other table:

```sql
create type pet_typ as object (
    id number,
    name varchar2(50),
    type varchar2(50),
    owner_id number
);
/

create table objtable_person4 of person_typ (id primary key)
object identifier is primary key;
/

create table objtable_pet of pet_typ (id primary key)
object identifier is primary key;
/

alter table objtable_pet
add constraint "PERSON_PET_FK" FOREIGN KEY ("OWNER_ID") REFERENCES "OBJTABLE_PERSON4"("ID");
/

insert into objtable_person4 values (person_Typ(1, 'John', 'Smith', NULL));
insert into objtable_pet values (pet_typ(1, 'Spot', 'Dog', 1));
```

An alternate syntax is to embed it all in the create table statement, similar to:

```sql
create table objtable_pet2 of pet_typ (
primary key(id),
foreign key (owner_id) references objtable_person4
)
```

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

    l_person := person_typ(NULL,NULL,NULL,NULL);
    test_null(l_person);--Not NULL


end;
/
```

So far, the examples have only been using a type specification, where to initialise the the object you need to have a value for each attribute. e.g. `l_person := person_typ('John', 'Smith', to_date('01-01-2001', 'DD-MM-YYY'))`. We can now extend on this, to introduce constructors and methods against the object.

They can be classed into:

* Member functions/procedures
* Static functions/procedures
* Constructors

## Member functions/procedures

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

By default, an object has an implicit constructor, which expects a value for each defined field. The previous examples have demonstrated this with `person_typ(NULL,NULL,NULL,NULL)` - setting a NULL value for each field. Suppose however, that you want to support instantiation of your object without passing in the hire date (e.g. just passing in the first name and last name).

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


# Object Inheritance

PL/SQL types support parent-child relationships, as you'd expect from an Object-Oriented language. We already have a person_typ, so suppose we wanted to expand on this to have object for particular types of people.

Objects have a property that determine whether or not they can be extended on. That is whether it is `FINAL` or `NOT FINAL`. Only those objects that explictly specify that they are `NOT FINAL` can be inherited in any sub-types. On a subtype, you do not specify the `is object` clause, but rather `under parent` where parent is the parent type you are inheriting from. You specify whether a type is final at the end of the specification after all the fields and attributes; alternatively, you can alter the type to make it final or not final.

```plsql
create or replace type house_typ is object (
    age NUMBER,
    rooms NUMBER
) NOT FINAL;
/
--or
alter type house_typ NOT FINAL;
/
```

Once an object has dependents, it will not be possible to compile the type specification anymore, as it is relied on by other objects. You can however issue alter statements (with the `cascade` or `invalidate` options) to the type to add new fields. note: This rule doesn't apply to type bodies - they can be recompiled without issue.

```plsql
create or replace type person_typ3 is object (
    id NUMBER,
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE
) NOT FINAL;
/

create or replace type developer_typ under person_typ3  (
    programming_language varchar2(50)
);
/

--can only add new fields to person_typ with the alter type statement
alter type person_typ add attribute termination_date DATE cascade;
/
```

With this, our new `developer_typ` object automatically contain all the fields contained in `person_typ`. The default constructor then follows the field in the order from top level, down.

```plsql
declare
    l_dev developer_typ;
begin
    --developer_typ(ID, FIRST_NAME, LAST_NAME, HIRE_DATE, TERMINATION_DATE, PROGRAMMING_LANGUAGE)
    l_dev := developer_typ(1, 'John', 'Smith', NULL, NULL, 'Java');

    dbms_output.put_line(l_dev.programming_language);--Java
end;
```

If you wish to override a function/procedure in a subtype (having a function/procedure with the same signature as its parent), you need to prefix the function/procedure with the `overriding`clause. In the subtype, if you wish to call a parents function, you need to cast self as the parent type (which is valid, since it inherits), then call the function/procedure.

```plsql
create or replace type person_typ is object (
    id NUMBER,
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE,
    member procedure print_info
) NOT FINAL;
/

create or replace type body person_typ as

    member procedure print_info
    as
    begin
        dbms_output.put_line(self.first_name || ' ' || self.last_name);
    end;

end;
/

create or replace type developer_typ under person_typ  (
    programming_language varchar2(50),
    overriding member procedure print_info
);
/

create or replace type body developer_typ as

    overriding member procedure print_info
    as
    begin
        (self as person_typ).print_info();
        dbms_output.put_line(self.programming_language);
    end print_info;

end;
/
```

The neat thing with this type inheritance is that you can create an object table referring to the parent and any subtypes are able to be stored in the table. Although, just querying the table you would only get the attributes from the parent type.

```plsql
create or replace type person_typ is object (
    id NUMBER,
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE,
    member function get_info return varchar
) NOT FINAL;
/

create or replace type body person_typ as

    member function get_info return varchar
    as
    begin
        return self.first_name || ' ' || self.last_name;
    end;

end;
/

create or replace type developer_typ under person_typ  (
    programming_language varchar2(50),
    overriding member function get_info return varchar
);
/

create or replace type body developer_typ as

    overriding member function get_info return varchar
    as
    begin
        return (self as person_typ).get_info() || ' --Language: ' || self.programming_language;
    end get_info;

end;
/
--

create table objtable_people of person_Typ;
/

insert into objtable_people values (person_typ(1, 'John', 'Smith', NULL));
insert into objtable_people values (developer_typ(2, 'Mark', 'Henderson', NULL, 'PL/SQL'));
/

select otp.*, otp.get_info() person_info
from objtable_people otp;
/


/* Query Result:
ID   FIRST_NAME  LAST_NAME  HIRE_DATE  PERSON_INFO
---  ----------- ---------- ---------  -------------------------------
 1   John        Smith                 John Smith
 2   Mark        Henderson             Mark Henderson --Language: PL/SQL   
*/
```

You may like to make your types or functions abstract, that is making them so they cannot be instantiated. To do so, you can specify `NOT INSTANTIABLE` then only subclasses that inherit will be able to be initialised in your programs..

```plsql
create or replace type person_typ is object (
    id NUMBER,
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE,
    member function get_info return varchar2
)
NOT INSTANTIABLE
NOT FINAL;
/

create or replace type body person_typ
as

    member function get_info return varchar2
    as
    begin
        return self.first_name;
    end get_info;

end;
/

create or replace type developer_typ under person_typ(
    programming_language varchar2(50)
);
/

declare
    l_pers person_typ;

begin

    l_pers := person_typ(1, 'John', 'Smith', NULL);
    dbms_output.put_line(l_pers.get_info());--never get here
    /*
    ORA-06550: line 6, column 15:
    PLS-00713: attempting to instantiate a type that is NOT INSTANTIABLE
    ORA-06550: line 6, column 5:
    PL/SQL: Statement ignored
    06550. 00000 -  "line %s, column %s:\n%s"
    *Cause:    Usually a PL/SQL compilation error.
    *Action:
    */

end;
/

declare
    l_pers developer_typ;
begin

    l_pers := developer_typ(1, 'John', 'Smith', NULL, 'PL/SQL');
    dbms_output.put_line(l_pers.get_info());--John

end;
/
```

In the case of applying `NOT INSTANTIABLE` to a method, this serves as as a mechanism to declare a method specification, without implementing it in the type body (your subtypes would each have their own implementation). If you declare a method in the type specification, you would usually have to add an implementation.

```plsql
create or replace type person_typ is object (
    id NUMBER,
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE,
    not instantiable member function get_info return varchar2,
    member function get_id return NUMBER
)
NOT INSTANTIABLE
NOT FINAL;
/

create or replace type body person_typ
as

    member function get_id return NUMBER
    as
    begin
        return self.id;
    end;    

end;
/
```

You may also wish to restrict that a method can't be overridden in a future subtype. This is achievable my adding the `FINAL` property to the start of the method signature.

The final thing you will likely want to be able to do is test the type of an object. The predicate `is of type` will be useful here. In th example, I am outputing a string depending on the type - following this, you will want to start with a bottom-up approach, since if you do a top-down approach all cases will match the first test, since all developers are people.

```sql
create or replace type person_typ is object (
    id NUMBER,
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE,
) NOT FINAL;
/

create or replace type developer_typ under person_typ  (
    programming_language varchar2(50)
);
/

create table objtable_people of person_Typ;
/

insert into objtable_people values (person_typ(1, 'John', 'Smith', NULL));
insert into objtable_people values (developer_typ(2, 'Mark', 'Henderson', NULL, 'PL/SQL'));
/

select
    otp.id
  , otp.last_name
  , case
        when value(otp) is of (developer_typ)
            then 'Developer'
        when value(otp) is of (hr.person_typ)
            then 'Person'

    end type_kind
from objtable_people otp

/* Output:

ID  LAST_NAME   TYPE_KIND
--- ----------- --------------
 1  Smith       Person   
 2  Henderson   Developer
 */
```

We can also use what we've learnt here for use in our PL/SQL code, with the addition of the `treat` function in order to cast what is stored into sub-types.

```plsql
create or replace type person_typ is object (
    id NUMBER,
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE
) NOT FINAL;
/

create or replace type developer_typ under person_typ  (
    programming_language varchar2(50)
);
/

create table objtable_people of person_Typ;
/

insert into objtable_people values (person_typ(1, 'John', 'Smith', NULL));
insert into objtable_people values (developer_typ(2, 'Mark', 'Henderson', NULL, 'PL/SQL'));
/

declare

    type t_person_list is table of person_typ
        index by PLS_INTEGER;
    l_all_people t_person_list;    

    l_person person_typ;
    l_developer developer_typ;

begin

    select value(otp)
    bulk collect into l_all_people
    from objtable_people otp;

    for i in 1..l_all_people.COUNT
    loop

        dbms_output.put_line('Row ' || i);

        if l_all_people(i) is of (developer_typ)
        then
            dbms_output.put_line('Developer');
            l_developer := treat(l_all_people(i) as developer_typ);

            dbms_output.put_line('First name: ' || l_developer.first_name);
            dbms_output.put_line('Last name: ' || l_developer.last_name);
            dbms_output.put_line('Programming Language: ' || l_developer.programming_language);
        elsif l_all_people(i) is of (person_typ)
        then
            dbms_output.put_line('Person');
            l_person := treat(l_all_people(i) as person_typ);
            dbms_output.put_line('First name: ' || l_person.first_name);
            dbms_output.put_line('Last name: ' || l_person.last_name);
        else
            dbms_output.put_line('Uknown type');
        end if;

    end loop;

    /* Output:


    Row 1
    Person
    First name: John
    Last name: Smith
    Row 2
    Developer
    First name: Mark
    Last name: Henderson
    Programming Language: PL/SQL
    */

end;
/
```
