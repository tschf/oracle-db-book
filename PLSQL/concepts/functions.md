# Functions

## Calling from SQL

A function is a PL/SQL subprogram that will return a value. As with any sub-program, you can perform PL/SQL or SQL operations. However, if you are calling a function from a DML operation, there are some limitations that you should keep in mind.

In a `select` statement, you cannot issues DML operations that will modify any database tables. For example, suppose we had the bright idea to log any random number we generate in our function `get_Random_num`

```plsql
create table random_log(
    ID NUMBER PRIMARY KEY,
    RAND_VAL NUMBER NOT NULL);
/

create sequence random_log_seq;
/


create or replace function get_random return number
as
    l_random_num NUMBER;
begin
    l_random_num := floor(DBMS_RANDOM.VALUE(1,20));

    insert into random_log values (random_log_seq.nextval, l_random_num);

    return l_random_num;
end get_random;

select employee_id, last_name, get_random random_num
from employees
where employee_id  = 100
```
Would not work, because we cannot modify table data from within the called function from an SQL statement.

```
Error report -
SQL Error: ORA-14551: cannot perform a DML operation inside a query
ORA-06512: at "HR.GET_RANDOM_NUM", line 7
14551. 00000 -  "cannot perform a DML operation inside a query "
*Cause:    DML operation like insert, update, delete or select-for-update
           cannot be performed inside a query or under a PDML slave.
*Action:   Ensure that the offending DML operation is not performed or
           use an autonomous transaction to perform the DML operation within
           the query or PDML slave.
```

Another limitation that you need to be cautious of is when performing updates on a series of rows, you can not necessarily fetch rows from the same table, since the data is changing as you perform each update. You will more than likely get the mutating table error.

```plsql
create or replace function get_random_empdate return DATE
as
    l_random_num NUMBER;
    l_max_rows NUMBER;
    l_hire_date DATE;
begin

    select count(1)
    into l_max_rows
    from employees;

    l_random_num := floor(DBMS_RANDOM.VALUE(1,l_max_rows));

    with emp_data as (
        select hire_date, row_number() over (order by hire_date desc) rn
        from employees
    )
    select hire_date
    into l_hire_date
    from emp_data
    where rn = l_random_num;

    return l_hire_date;
end get_random_empdate;
/

begin
    for i in 1..10
    loop
        update employees
        set hire_date = get_random_empdate
        where employee_id = 100+i;
    end loop;

end;
```

Output:
```
Error report -
ORA-04091: table HR.EMPLOYEES is mutating, trigger/function may not see it
ORA-06512: at "HR.GET_RANDOM_EMPID", line 8
ORA-06512: at line 4
04091. 00000 -  "table %s.%s is mutating, trigger/function may not see it"
*Cause:    A trigger (or a user defined plsql function that is referenced in
           this statement) attempted to look at (or modify) a table that was
           in the middle of being modified by the statement which fired it.
*Action:   Rewrite the trigger (or function) so it does not read that table
```
