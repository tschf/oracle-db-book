# Bulk processing

Bulk processing is a way you can make your PL/SQL programs perform better. When you execute DML statements in your programs, you need to do a switch between the PL/SQL engine and the SQL engine. When you do a lot of these switches, and can cause your programs to run quite slow. Bulk processing allows you to perform these statements in batches, thus performing less switches between the two engines.

The two statements that allow you to do bulk processing are:

1. bulk collect
* forall

## Bulk collect

`bulk collect` allows you to fetch the results of a particular collection into an associative array. You will need to either declare a new record type that the data can be returned to, or base it off a table record type:

```plsql
declare
    type t_emp_list is table of employees%rowtype
        index by pls_integer;
    l_emps t_emp_list;
begin
    select *
    bulk collect into l_emps
    from employees;

    dbms_output.put_line('Bulk collect fetched: ' || l_emps.COUNT || ' rows.');

end;
```

If you don't directly query from a table, you can create a custom record type.

```plsql
declare
    type t_emp_info is record (
        employee_id employees.employee_id%type,
        salary employees.salary%type
    );
    type t_emp_list is table of t_emp_info
        index by pls_integer;
    l_emps t_emp_list;
begin
    select employee_id, salary
    bulk collect into l_emps
    from employees;

    dbms_output.put_line('Bulk collect fetched: ' || l_emps.COUNT || ' rows.');

end;
```

This also works with the `returning` clause. For instance, you can return any rows that have been affected by an update statement into a collection. Suppose you want to do another operation on only the rows that were affected by an update statement, you could return any needed data into a collection.

```plsql
declare
    type t_affected_emp_ids is table of employees.employee_id%type
        index by PLS_INTEGER;
    l_affected_rows t_affected_emp_ids;
begin

    update employees
    set salary = salary*2
    where employee_id in (101, 102, 103)
    returning employee_id bulk collect into l_affected_rows;

    dbms_output.put_line(l_affected_rows.COUNT || ' rows affected by the update statement');

end;
```

## Forall

`forall` send dml operations in batches to the SQL engine. One difference between a regular for loop and a forall loop is that a a forall loop can only contain one statement - there is no `loop` or `end loop` cause on the `forall` block.

```plsql
declare
    type t_emps is table of employees%rowtype;
    l_emps t_emps;
begin

    select *
    bulk collect into l_emps
    from employees;

    forall i in 1..l_emps.COUNT
        update employees
        set salary = salary*2
        where employee_id = l_emps(i).employee_id;

end;
```

You can only use a forall loop when you use the loop variable to lookup a value in an collection.

```plsql
create table bulk_demo(id NUMBER primary key);
/

begin


    --forall i in 1..2
    --    insert into employees (employee_id, last_name, email, hire_date, job_id)
    --    values (100+i, 'Test'||i, i, sysdate, 'AD_PRES');

    forall i in 1..2
        insert into bulk_demo (id)
        values (i);

end;
/
```

Output:
```
ORA-06550: line 10, column 17:
PLS-00430: FORALL iteration variable I is not allowed in this context
ORA-06550: line 9, column 9:
PLS-00435: DML statement without BULK In-BIND cannot be used inside FORALL
06550. 00000 -  "line %s, column %s:\n%s"
*Cause:    Usually a PL/SQL compilation error.
*Action:
```

However, if we instead lookup a value from a collection (varray in this example), it will work.

```plsql
create table bulk_demo2 (id number primary key);
/

declare
    type t_list is varray(2) of number;
    l_nums t_list := t_list(1,7);
begin


    forall i in 1..2
        insert into bulk_demo2 (id)
        values (l_nums(i));

end;
/
```

In a traditional loop, you can wrap any dml operations in a block so that you can handle exceptions on a row-by-row basis. This is not possible in a forall loop since you can only have one dml statement in the body of the forall statement.

The way you can handle this is to use the `save exceptions` clause on the `forall` specification. This then returns the errors raised into the cursor variable `SQL%BULK_ERRORS`, which is a collection of records with the fields:

1. error_index
* error_code

You need to handle this in an exception block associated to the error code `-24381`.

```plsql
create table bulk_demo3 (id number primary key);
/

declare
    type t_list is varray(2) of number;
    l_nums t_list := t_list(1,1);

    bulk_errors exception;
    pragma exception_init(bulk_errors, -24381);
begin

    forall i in 1..2 save exceptions
        insert into bulk_demo3 (id)
        values (l_nums(i));

    exception
    when bulk_errors then
        dbms_output.put_line('Could not insert: ' || SQL%BULK_EXCEPTIONS.COUNT || ' row(s)');
end;
```

Output:
```
Could not insert: 1 row(s)
```

We can also see how many rows were affected by the regular SQL%ROWCOUNT cursor variable. In addition, we get access to an associate array cursor variable SQL%BULK_ROWCOUNT which shows how many rows were affected in each sequential DML statement in the forall loop.

```plsql
create table bulk_demo4(
    id number primary key,
    dept_no number not null);
/

insert into bulk_demo4 values (1, 10);
insert into bulk_demo4 values (2, 10);
insert into bulk_demo4 values (3, 20);
insert into bulk_demo4 values (4, 20);
insert into bulk_demo4 values (5, 20);
insert into bulk_demo4 values (6, 30);
insert into bulk_demo4 values (7, 30);
insert into bulk_demo4 values (8, 30);
insert into bulk_demo4 values (9, 30);
insert into bulk_demo4 values (10, 40);
/

declare
    type t_depts is varray(4) of number;
    l_depts t_depts := t_depts(10,20,30,40);
begin

    forall i in l_depts.FIRST..l_depts.LAST
        delete from bulk_demo4
        where dept_no = l_depts(i);

    dbms_output.put_line('Total affected rows: ' || SQL%ROWCOUNT);

    for i in l_depts.FIRST..l_depts.LAST
    LOOP

        dbms_output.put_line('dept ' || l_depts(i) || ' affected rows: ' || SQL%BULK_ROWCOUNT(i));

    END LOOP;


end;
```

Output:
```
Total affected rows: 10
dept 10 affected rows: 2
dept 20 affected rows: 3
dept 30 affected rows: 4
dept 40 affected rows: 1
```
