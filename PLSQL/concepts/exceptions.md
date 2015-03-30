# Exceptions

A PL/SQL program's block structure is split into three area - the declaration section, where variables are defined. The execution block which is where the business logic is implemented, the last part is handling any exceptions that may have occurred. With the exception block being the last bit of any given block (that is, it can't appear in the middle - that is, without declaring a sub-block).

Exception handlers allow you to properly handle exceptions that may arise, and log any information you may need, as well as feed back more useful error messages so that you can properly associate what has gone wrong. For example, in a typical row lookup, you may like to feedback some more information about any parameters passed in.

```
set serveroutput on

create or replace function get_last_name(p_id in employees.employee_id%type) return employees.last_name%type
as
    l_last_name employees.last_name%type;
begin

    select last_name into l_last_name
    from employees
    where employee_id = p_id;

    return l_last_name;

    exception
        when no_data_found
        then
            dbms_output.put_line('Could not find employee with id: "' || p_id || '"');
            raise;


end get_last_name;
/

begin
    dbms_output.put_line(get_last_name(1));
end;
/
```

Output:
```
Error report -
ORA-01403: no data found
ORA-06512: at "HR.GET_LAST_NAME", line 16
ORA-06512: at line 3
01403. 00000 -  "no data found"
*Cause:    No data was found from the objects.
*Action:   There was no data from the objects which may be due to end of fetch.
Could not find employee with id: "1"
```

Sometimes exceptions don't have a defined name. This is quite typical when using the `raise_application_error` function which allows you to raise your own errors as well as provide a customised message.

One way to handle this is to test the `sqlcode` in the `others` exception section

```plsql
begin

    raise_application_Error(-20500, 'Demonstration of raising an exception', true);

exception
    when others
    then
        if sqlcode = -20500
        then

            dbms_output.put_line('Handling an exception');
        else
            raise;
        end if;
end;
```

A better approach is to use the `exception_init` pragma to associate an error code to a named exception.

```plsql
declare
    e_custom_exc exception;
    pragma exception_init(e_custom_exc, -20500);
begin

    raise_application_error(-20500, 'Demonstration of raising an exception');

    exception
    when e_custom_exc
    then
        dbms_output.put_line('Handling an exception');
end;
```

Exceptions can be explicitly risen by the `raise` statement, followed by the name of the exception - which can be a user-defined exception or a global exception. To ensure an exception is propagated back through calling code, you can also re-raise an exception in an exception block without the need to specify the type of exception being risen - that is, with the `raise` clause on its own.
