# User-defined types

PL/SQL offers two kinds user defined types that you can declare at either the schema or block level:

1. Records
* Objects

## Record

A record is a custom type that you can define, that can contain one or more fields. It can only be declared at the block level. If you want to expose records for other programs to use, you can declare it in the package specification level.

You general syntax for creating a record type is: type _typeName_ is record (_fieldList fieldDataType_)

For example, suppose we wanted a record to store a persons first name, last name, and date of birth, we could define it like so:

```plsql
declare
    type t_person_rec is record (
        first_name varchar2(50),
        last_name varchar2(50),
        date_of_birth DATE
    );
    l_person t_person_rec;
begin
    --do stuff with l_person
end;
```

When you declare a variable that is a record, each field defined in the record type is automatically assigned a value of `NULL`.

```plsql
declare
    type t_person_rec is record (
        first_name varchar2(50),
        last_name varchar2(50),
        date_of_birth DATE
    );
    l_person t_person_rec;
begin

    DBMS_OUTPUT.PUT_LINE('First name: ' || coalesce(l_person.first_name, 'Not defined'));
    DBMS_OUTPUT.PUT_LINE('Last name: ' || coalesce(l_person.last_name, 'Not defined'));
    DBMS_OUTPUT.PUT_LINE('DOB: ' || coalesce(to_char(l_person.date_of_birth), 'Not defined'));

end;
```
Output:
```
anonymous block completed
First name: Not defined
Last name: Not defined
DOB: Not defined
```

If you need to define a record based off one of your tables, it is not necessary to delcare a new type and duplicate all the fields - which can be error prone in case any fields change. Here you can make use of the `%rowtype` attribute, which refers to the specification of the table you call it with. The syntax is: _tableName_%rowtype

```plsql
declare
    l_emp employees%rowtype;
begin
    select *
    into l_emp
    from employees
    where employee_id = 101;

    dbms_output.put_line('Employee id: ' || l_emp.employee_id);
    dbms_output.put_line('First name: ' || l_emp.first_name);
    dbms_output.put_line('Last name: ' || l_emp.last_name);
    --other columns removed
    dbms_output.put_line('Manager id: ' || l_emp.manager_id);
    dbms_output.put_line('Department id: ' || l_emp.department_id);

end;
```

Output:
```
Employee id: 101
First name: Neena
Last name: Kochhar
Manager id: 100
Department id: 90
```

What's also handy, when dealing with records using `%rowtype` of a table, and the table does not have any virtual columns, you can pass the record variable in its entirety into the values portion of the insert statement to add that data.

```plsql
create table insert_from_record(
    ID NUMBER,
    AGE NUMBER,
    CREATED_DATE DATE);
/

declare
    l_rec insert_from_Record%rowtype;
begin
    l_Rec.ID := 1;
    l_rec.AGE := 55;
    l_rec.created_date := sysdate;

    insert into insert_from_record values l_rec;
end;

select *
from insert_from_Record
```
Output:
```
        ID        AGE CREATED_DATE
---------- ---------- ------------
         1         55 15/MAR/15

```

Any you can do an update using the `row` keyword.

```plsql
declare

    l_rec insert_from_record%rowtype;

begin
    l_Rec.ID := 1;
    l_rec.AGE := 60;

    update insert_from_record
    set row = l_Rec
    where id = l_rec.id;

end;
```

You cannot perform comparisons on two record variables, however if two record variables have a matching specification, you can assign one record to another.

## Object

An object is the other type that can be created and is much more extensible than a record - as well as having slightly different syntax.

Unlike records, objects can not be declared in-line (for procedural blocks of code) - but need to be declared at the schema level.

```sql
create or replace type t_person_obj is object (
    first_name varchar2(20),
    last_name varchar2(25),
    hire_date DATE
);
/
```

When we want to use this object in a PL/SQL code block, we need to first initialize it. That is, we can't follow the pattern of `record`s and just assign values without first initialising the object.

If we try to run the following code:

```plsql
declare
    l_info t_person_obj;
begin
    l_info.first_name := 'Thomas';
end;
```

we get the following error:

```
Error report -
ORA-06530: Reference to uninitialized composite
ORA-06512: at line 4
06530. 00000 -  "Reference to uninitialized composite"
*Cause:    An object, LOB, or other composite was referenced as a
           left hand side without having been initialized.
*Action:   Initialize the composite with an appropriate constructor
           or whole-object assignment.
```

So, we would need to initialize the object. Our object has three properties (first_name, last_name and hire_date) so if we don't know the values at initialisation time, we can just pass NULL.

In this simplistic set up of the object, we **must** pass in the number of parameters that exist in the object.

```plsql
declare
    l_person1 t_person_obj;
    l_person2 t_person_obj;

    procedure print_person_info(p_person in t_person_obj)
    as
    begin
        dbms_output.put_line('First: ' || p_person.first_name);
        dbms_output.put_line('Last: ' || p_person.last_name);
        dbms_output.put_line('Hire Date: ' || p_person.hire_date);
        dbms_output.put_line('..');
    end print_person_info;
begin
    l_person1 := t_person_obj(NULL,NULL,NULL);

    l_person1.first_name := 'Thomas';
    l_person1.last_name := 'Smith';
    l_person1.hire_date := date '1946-01-03';

    print_person_info(l_person1);
    --

    l_person2 := t_person_obj('Mary-Jane', 'Smith', date '1949-05-05');

    print_person_info(l_person2);
end;
/
```

Output:
```
First: Thomas
Last: Smith
Hire Date: 03/JAN/46
..
First: Mary-Jane
Last: Smith
Hire Date: 05/MAY/49
..
```

Unlike records where we can select table columns directly into the user-defined type, when working with objects you need to wrap the matching columns in the objects constructor like so:

```plsql
declare
    l_person t_person_obj;

    procedure print_person_info(p_person in t_person_obj)
    as
    begin
        dbms_output.put_line('First: ' || p_person.first_name);
        dbms_output.put_line('Last: ' || p_person.last_name);
        dbms_output.put_line('Hire Date: ' || p_person.hire_date);
        dbms_output.put_line('..');
    end print_person_info;
begin
    select t_person_obj(first_name, last_name, hire_Date)
    into l_person
    from employees
    where employee_id = 101;

    print_person_info(l_person);

end;
/
```

Output:
```
First: Neena
Last: Kochhar
Hire Date: 21/SEP/05
..

```
