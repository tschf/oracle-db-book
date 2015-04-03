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
