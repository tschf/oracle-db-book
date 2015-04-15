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

## Re-usable cursors (ref cursors)

When you define a type that is a `ref cursor`, you can set it as the data type for variables (similar to how you do _cursorName_%type). You can optionally specify a return type of the `ref cursor` which means if you try and re-assign the value, it must conform to that record type.

There is a built in ref cursor type that you can use, `sys_refcursor`, so that you don't have to declare your own `ref cursor` type.

You can then fetch data using this ref cursor passing either a query or a string containing a query to the `open for` clause of the cursor variable.

```plsql
set serveroutput on

declare
    l_curs sys_refcursor;
    l_emp employees%rowtype;

    l_dyn_curs_query varchar2(200) := 'select * from employees where employee_id in (101,102)';
begin

    dbms_output.put_line('Pass a query');
    open l_curs for
        select *
        from employees
        where employee_id in (101, 102);

    loop
        fetch l_curs into l_emp;
        exit when l_curs%NOTFOUND;
        dbms_output.put_line(l_emp.last_name);
    end loop;

    close l_curs;--not strictly necessary, the resources for this cursor will be lost when re-opening it again
    dbms_output.put_line('');
    dbms_output.put_line('Pass a string containing a query');
    open l_curs for
        l_dyn_curs_query;

    loop
        fetch l_curs into l_emp;
        exit when l_curs%NOTFOUND;
        dbms_output.put_line(l_emp.last_name);
    end loop;

    close l_curs;


end;
```

Output
```
Pass a query
Kochhar
De Haan

Pass a string containing a query
Kochhar
De Haan
```

In the above snipped, we used the `sys_refcursor` data type, but could have easily declared our own type.

```plsql
type t_generic_cursor is ref cursor;
l_curs t_generic_cursor
```

When declaring our own `ref cursor` type, we can make it strongly typed by giving it a return type, which enforces that when you open the cursor, the query you pass in must return that record type. You also **can not** pass a query in the form of a string.

Ref cursors are useful also because you can passed them as named parameters to sub programs (of course, collecting the data into a collection and passing the collection is the other option).

```plsql
declare
    type t_employee_curs is ref cursor return employees%rowtype;
    l_curs t_employee_curs;

    procedure print_last_names(p_emp_data in t_employee_curs)
    as
        l_emp employees%rowtype;
    begin
        loop
            fetch p_emp_data into l_emp;
            exit when p_emp_data%NOTFOUND;
            dbms_output.put_line(l_emp.last_name);
        end loop;
    end print_last_names;
begin

    open l_curs for
        select *
        from employees
        where employee_id in (101,102);

    print_last_names(l_curs);

    close l_curs;
end;
```

## Nested cursors

The `cursor` expression allows you to nest a subquery to return a second cursor for each row. When you fetch the record, you fetch the nested `cursor` into a `ref cursor` object, that you can then fetch each row for.

```plsql
declare


    type t_city is record ( city varchar2(30) );
    type t_city_rc is ref cursor return t_city;

    cursor c_countries is
        select country_name, cursor(select city from locations where country_id = countries.country_id)
        from countries
        where country_id = 'IT';

    l_country countries.country_name%type;
    l_city_cur t_city_rc;
    l_city t_city;

begin

    open c_countries;
    null;
    loop
        fetch c_countries into l_country, l_city_cur;
        exit when c_countries%notfound;

        dbms_output.put_line(l_country);
        dbms_output.put_line('Locations: ');
        loop
            fetch l_city_cur into l_city;
            exit when l_city_cur%NOTFOUND;
            dbms_output.put_line(l_city.city);
        end loop;
        dbms_output.put_line('');
    end loop;

end;
```
Output:
```
Italy
Locations:
Roma
Venice
```

## Cursor attributes

* %ISOPEN
* %FOUND
* %NOTFOUND
* %ROWCOUNT
* %BULK_ROWCOUNT
* %BULK_EXCEPTIONS

For implicit cursors, each attribute is accessed with the `SQL` keyword, otherwise they are accessed with the named cursor. If you are using a cursor variable (ref cursor), attempting to access any attribute other than %ISOPEN will throw the `INVALID_CURSOR` exception.

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

Returns a collection containing the number of rows affected by each sequential statement in a `forall` loop.

```plsql
declare

    type t_2array is varray(2) of departments.department_id%type;
    l_rows t_2array := t_2array(10, 20);

begin

    forall i in l_rows.first..l_rows.last
        update employees
        set salary = salary*1.1
        where department_id = l_rows(i);


    for i in 1..l_rows.last
    LOOP
        dbms_output.put_line('Department: ' || l_rows(i) || ' update affected ' || SQL%BULK_ROWCOUNT(i) || ' rows');
    END LOOP;

end;
```

### SQL%BULK_EXCEPTIONS

When specifying `save exceptions` on the `forall` loop, any exceptions are returned into this cursor variable. Each element is a record containing an error code and error index.

```plsql
create table bulk_insert(
    id number primary key,
    str varchar2(2)
);
/

set serveroutput on

declare

    type t_3array is varray(3) of departments.department_id%type;
    l_rows t_3array := t_3array(1,2,3);

    bulk_Exception exception;
    pragma exception_init(bulk_Exception, -24381);
begin

    forall i in l_rows.first..l_rows.last save exceptions
        insert into bulk_insert (id, str)
        values (l_rows(i), dbms_Random.string('u', l_rows(i)));

exception
    when bulk_Exception
    then
        dbms_output.put_line('Errors: ' || SQL%BULK_EXCEPTIONS.COUNT);
        for i in 1..SQL%BULK_EXCEPTIONS.COUNT
        LOOP
            dbms_output.put_line('Error index: ' || SQL%BULK_EXCEPTIONS(i).error_index);
            dbms_output.put_line('Error code: ' || SQL%BULK_EXCEPTIONS(i).error_code);
        END LOOP;


end;
/
```
