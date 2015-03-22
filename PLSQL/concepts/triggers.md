# Triggers

Unlike other subprograms - functions, procedures - a trigger is a subprogram that isn't called directly, but called when certain events occur. Events can be DML related - insert, update, delete - DDL related - create, drop - or system level events such as log on, log off, start up, shutdown, etc.

## DML triggers

A trigger can be associated with either a table or view.

When creating a trigger you need to know if you want it to run `before` or `after` the associated event.

### Table trigger

For DML triggers, you specify the event on which it should fire and the table to which it applies to. The event can be a combination of `insert`, `update` or `delete`.

```
create or replace trigger t1_region
before insert or update or delete on regions
begin
    if inserting then
        dbms_output.put_line('Inserting into regions');
    elsif updating then
        dbms_output.put_line('Updating regions');
    elsif deleting then
        dbms_output.put_line('Deleting from regions');
    end if;
end;
/

insert into regions values (5, 'Oceani');
update regions set region_name = 'Oceania' where id = 5;
delete from regions where region_id = 5;
/
```
Output:
```
Inserting into regions
Updating regions
Deleting from regions
```

When specifying `update`, you can proceed it with `of <column list>` to only fire when updating any of the specified column list. Then in the body, you can pass a value to the `updating` clause to check if that particular column is being updated.

```plsql
create or replace trigger t1_region
before insert or update of region_name or delete on regions
begin
    if inserting then
        dbms_output.put_line('Inserting into regions');
    elsif updating('region_name') then
        dbms_output.put_line('Updating region_name');
    elsif deleting then
        dbms_output.put_line('Deleting from regions');
    end if;
end;
/

insert into regions values (5, 'Oceani');
update regions set region_name = 'Oceania' where region_id = 5;
update regions set region_id = 6 where region_id = 5;
delete from regions where region_id = 6;
/
```

Output:
```
Inserting into regions
Updating region_name
Deleting from regions
```

The examples thus far have been statement level triggers, they will only execute once per each operation. Suppose you issue an update statement that affects more than one row, you may like the trigger to fire once per affected row. To do so, you can add the `for each row` clause before the body of the trigger. The benefit of row level triggers is that you can also access new and old pseudo records which contain all fields in the table that is affected. On `before` triggers, it is possible to modify the `new` value.

```plsql
create or replace trigger t1_region
before update on regions
for each row
begin
    dbms_output.put_line ('Updating region name from "' || :old.region_name || '" to "' || :new.region_name || '"');
end;
/

update regions
set region_name = substr(region_name, 1, 10);
/
```

Output:
```
Updating region name from "Europe" to "Europe"
Updating region name from "Americas" to "Americas"
Updating region name from "Asia" to "Asia"
Updating region name from "Middle East and Africa" to "Middle Eas"
```

If you don't like the pseudorecord names of `new` and `old` you can give then a new name with the `referencing` clause after the table name liks so.

```plsql
create or replace trigger t1_region
before update on regions
referencing
    new as modified
    old as original
for each row
begin
    dbms_output.put_line ('Updating region name from "' || :original.region_name || '" to "' || :modified.region_name || '"');
end;
```

### View trigger

To create a trigger on a view, you need to use an `instead of` trigger because view is just a logical representation of one or more tables. You don't specify a `before` or `after` event like in a table trigger, but instead specify the clause `instead of <dml operation> on <view name>`. Much like a row level table trigger, it has access to the `new` and `old` pseudo records, but can not make any changes to the values.

```plsql
create or replace view v_emps as
select employees.employee_id, employees.last_name, employees.email, employees.hire_date, employees.job_id, departments.department_id, departments.department_name
from employees, departments
where employees.department_id = departments.department_id;
/

create or replace trigger t1_emp
instead of insert on v_emps
begin
    insert into employees
    (
          employee_id
        , last_name
        , email
        , hire_date
        , job_id
        , department_id
    )
    values
    (
          :new.employee_id
        , :new.last_name
        , :new.email
        , sysdate
        , :new.job_id
        , :new.department_id
    );
end t2_region;
/

insert into
v_emps (employee_id, last_name, email, job_id, department_id)
values (555, 'Rogerson', 'ROGERSON', 'ST_CLERK', 90);
```

And delete:

```plsql
create or replace view v_emps as
select employees.employee_id, employees.last_name, employees.email, employees.hire_date, employees.job_id, departments.department_id, departments.department_name
from employees, departments
where employees.department_id = departments.department_id;
/

create or replace trigger t2_emp
instead of delete on v_emps
begin
    delete from employees
    where employee_id = :OLD.employee_id;
end t2_region;
/

delete from v_emps where employee_id = 999;
/
```


### Compound DML triggers

//todo
