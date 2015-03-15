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

//todo
