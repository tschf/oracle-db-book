## Subprogram privileges

Every program unit has an `AUTHID` property which can be either set as `definer` (known as definer's right unit) or `current_user` (known as invoker's right unit). The default AUTHID for subprograms is `definer`, which we can see by querying the user_procedures view, on a brand new function:

```plsql
create or replace function foo return number
as
begin
    return 1;
end foo;
/

select object_name, authid
from user_procedures
where object_name = 'FOO';
```
Output
```
OBJECT_NAME  AUTHID
------------ ------------
FOO          DEFINER
```

You configure the authid of a subprogram by specifying either authid definer or authid current_user immediately before the declare block like so:

```plsql
create or replace function foo return number
authid current_user
as
begin
    return 1;
end foo;
/
```

Depending on the property value of `authid` the values of `current_schema` and `current_user` will vary when the subprogram is called. When `authid` is set to `definer` no matter where you call the program from, the value of `current_user` and `current_schema` will be that of the schema that the subprogram lives in. On the other hand, when `authid` is set to `current_user`, the value of `current_user` and `current_Schema` will be that of where the subprogram was called from.

**Definer**  

```plsql
create or replace function foo return number
authid definer
as
begin
    dbms_output.put_line('Current user: ' || sys_context('userenv', 'current_user'));
    dbms_output.put_line('Current schema: ' || sys_context('userenv', 'current_schema'));
    return 1;
end foo;
/
```
*hr*

```
select foo
from dual

       FOO
----------
         1

Current user: HR
Current schema: HR
```

*hr_limited*

```
select hr.foo
from dual

       FOO
----------
         1

Current user: HR
Current schema: HR
```

**current_user**  
```plsql
create or replace function foo return number
authid current_user
as
begin
    dbms_output.put_line('Current user: ' || sys_context('userenv', 'current_user'));
    dbms_output.put_line('Current schema: ' || sys_context('userenv', 'current_schema'));
    return 1;
end foo;
/
```
*hr*
```
select foo
from dual

       FOO
----------
         1

Current user: HR
Current schema: HR

```

*hr_limited*
```
select hr.foo
from dual

       FOO
----------
         1

Current user: HR_LIMITED
Current schema: HR_LIMITED
```

The value of `current_user` and `current_schema` has the obvious effect of what the function will be able to access (through it's grants and roles).

Suppose I had yet another schema, `hr_private` with some extra pay info that  the `hr` schema can see:

```sql
create table employee_bonus(
    ID NUMBER PRIMARY KEY,
    employee_id NUMBER,
    bonus_amount NUMBER,
    paid_date DATE);
/
insert into employee_bonus values (1, 101, 500, to_Date('01/01/2001', 'dd/mm/yyyy'));
insert into employee_bonus values (2, 101, 7600, to_Date('01/01/2002', 'dd/mm/yyyy'));
insert into employee_bonus values (3, 101, 200, to_Date('01/01/2003', 'dd/mm/yyyy'));
insert into employee_bonus values (4, 101, 1500, to_Date('01/01/2007', 'dd/mm/yyyy'));
commit;
/
grant select on employee_bonus to hr;
/
```

Then we have a function the returns the total bonus for an employee.

```plsql
create or replace function get_employee_bonus(p_employee_id employees.employee_id%type) return NUMBER
authid current_user
as
    l_total_amt NUMBER;
begin

    select sum(bonus_amount)
    into l_total_amt
    from hr_private.employee_bonus
    where employee_id = p_employee_id;

    return l_total_amt;

end get_employee_bonus;
/

grant execute on get_employee_bonus to hr_limited;
/
```

When we run that from the `hr` schema, we have no problem because it is running with `current_user` as `hr` which has select permissions on the table that is being queried. However, running from the `hr_limited` schema, which will set `current_user` to `hr_limited`

```
select get_employee_bonus(employee_id)
from employees
where employee_id = 101

ORA-00942: table or view does not exist
00942. 00000 -  "table or view does not exist"
*Cause:
*Action:
Error at Line: 2 Column: 6
```
