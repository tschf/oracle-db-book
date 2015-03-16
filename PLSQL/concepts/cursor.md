# Cursor

A `cursor` is a special type of object that represents information about a particular `DML` statement. Every time you perform a DML operation, the information stored is accessible by a set of attributes.

A cursor can be either `explicit` or `implicit`. The main difference is that with an `explicit` cursor, you declare it with the `cursor is` clause, and physically `open`, `fetch` rows, and `close` the cursor. On the other hand, implicit is just when the DML statement is inline in the body of the code. When accessing attributes, since an implicit cursor has no defined name, you reference attributes with the `SQL` keyword, wheras explicit you reference with the name of the cursor.

Implicit example

```plsql
begin

    update employees_demo
    set hire_date = sysdate
    where employee_id = 101;

    dbms_output.put_line(SQL%ROWCOUNT || ' row(s) affected');

end;
```

Output:
```
1 row(s) affected
```

Explicit example

```plsql
set serveroutput on

declare

    cursor emps is select first_name, manager_id from employees_demo where employee_id in (101, 102);
    emp_row emps%rowtype;

begin

    open emps;
    dbms_output.put_line(emps%ROWCOUNT || ' row(s) found');
    loop

        fetch emps into emp_row;
        exit when emps%NOTFOUND;
        dbms_output.put_line('First name: ' || emp_Row.first_name);
        dbms_output.put_line('Manager ID: ' || emp_Row.manager_id);
        dbms_output.put_line(emps%ROWCOUNT || ' row(s) found');


    end loop;
    close emps;
    --dbms_output.put_line(SQL%ROWCOUNT || ' row(s) found');

end;
```

Output:
```
0 row(s) found
First name: Neena
Manager ID: 100
1 row(s) found
First name: Lex
Manager ID: 100
2 row(s) found
```

## Cursor parameters

For an implicit cursor, passing a parameter is straight forward - you just pass the variable into the the query. However, explicit cursors need a bit more thought.

You can pass in any variables as you would with an implicit cursor, provided they are declared before the cursor. Whenever you open the cursor, the current value of any variables you pass in will be passed in.

The other option is to define a set of parameters that you can pass into cursor when opening. You do this by defining in parenthesis all the parameters you want available for the query against the cursor name. Then, when you open the cursor, you pass in any values that should be used.

```plsql
declare
    l_sal NUMBER;
    cursor emps is
        select *
        from employees
        where salary > l_sal;
    l_emp emps%rowtype;
begin
    l_sal := 20000;
    open emps;

    LOOP
        fetch emps into l_emp;
        exit when emps%NOTFOUND;
        dbms_output.put_line(l_emp.last_name || ' earns ' || l_emp.salary);
    END LOOP;

    close emps;

end;
```
or

```plsql
declare
    cursor emps (cp_sal NUMBER) is
        select *
        from employees
        where salary > cp_sal;
    l_emp emps%rowtype;  
    l_sal NUMBER;
begin
    l_sal := 20000;
    open emps(l_sal);

    LOOP
        fetch emps into l_emp;
        exit when emps%NOTFOUND;
        dbms_output.put_line(l_emp.last_name || ' earns ' || l_emp.salary);
    END LOOP;

    close emps;

end;
```

Output:
```
King earns 24000
```
## Looping

Looping over a cursor query is straight forward. An implicit is straight forward in that you just pass in the query to the `for` clause.

```plsql
begin
    for emp in (
        select * from employees
    )
    loop
        dbms_output.put_line(emp.last_name);
    end loop;
end;
```

For an explicit cursor, you do as the other examples have shown through this chapter. Open the cursor, then fetch rows row by row, until no more rows are found.

```plsql
declare
    cursor emps is select * from employees;
    emp emps%rowtype;

begin
    open emps;
    loop
        fetch emps into emp;
        exit when emps%NOTFOUND;
        dbms_output.put_line(emp.last_name);
    end loop;

    close emps;
end;
```

Or you can avoid the opening and closing the cursor by passing in the name of the cursor to the for loop, much like is done with an implicit cursor for loop.

```plsql
declare
    cursor emps is select * from employees;

begin
    for emp in emps
    loop
        dbms_output.put_line(emp.last_name);
    end loop;
end;
```

## Re-usable cursors

//todo

## Cursor attributes

* SQL%ISOPEN
* SQL%FOUND
* SQL%NOTFOUND
* SQL%ROWCOUNT
* SQL%BULK_ROWCOUNT
* SQL%BULK_EXCEPTIONS

### SQL%ISOPEN

Returns a BOOLEAN `true` or `false` depending if the cursor is currently open. The cursor is immediately closed in the case of an implicit cursor so will always return `false`.

```plsql
begin

    update employees_demo
    set hire_date = sysdate
    where employee_id = 101;

    dbms_output.put_line(case when SQL%ISOPEN then 'Cursor Open' else 'Cursor Closed' end);

end;
```

### SQL%FOUND

Returns a BOOLEAN `true` (if rows were found/affected) or `false` (if no rows were found/affected) depending on if any rows were found during the running of the SQL statement (or `null` if no statements have been run).

```plsql
set serveroutput on

begin

    update employees_demo
    set hire_date = sysdate
    where employee_id = 101;

    dbms_output.put_line(
        case
            when SQL%FOUND
                then 'Rows affected'
            else
                'No rows affected'
        end
    );

    update employees_demo
    set hire_date = sysdate
    where employee_id = 22;

    dbms_output.put_line(
        case SQL%FOUND
            when true
                then 'Rows affected'
            else
                'No rows affected'
        end
    );


end;
```

Output:
```
No rows affected
Rows affected
No rows affected
```

### SQL%NOTFOUND

Returns a BOOLEAN `true` (if no rows were found/affected) or `false` (if rows were found/affected) depending on if any rows were found during the running of the SQL statement (or `null` if no statements have been run).

```plsql
begin

    update employees_demo
    set hire_date = sysdate
    where employee_id = 55;

    dbms_output.put_line(
        case
            when SQL%NOTFOUND
                then 'No rows affected'
            else
                'Rows affected'
        end
    );

    update employees_demo
    set hire_date = sysdate
    where employee_id = 101;

    dbms_output.put_line(
        case
            when SQL%NOTFOUND
                then 'No rows affected'
            else
                'Rows affected'
        end
    );


end;
```

### SQL%ROWCOUNT

Returns a PLS_INTEGER of the number of rows that were found/affected by the last DML statement.

```plsql
declare
    l_employee employees_demo%rowtype;
begin

    select *
    into l_employee
    from employees_demo
    where employee_id = 101;

    dbms_output.put_line(SQL%ROWCOUNT || ' row(s) were found/affected');

    update employees_demo
    set hire_date = sysdate
    where employee_id = 55;

    dbms_output.put_line(SQL%ROWCOUNT || ' row(s) were found/affected');

    update employees_demo
    set hire_date = sysdate
    where employee_id = 101;

    dbms_output.put_line(SQL%ROWCOUNT || ' row(s) were found/affected');


end;
```

### SQL%BULK_ROWCOUNT

//todo

### SQL%BULK_EXCEPTIONS

//todo
