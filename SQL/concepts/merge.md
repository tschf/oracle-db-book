#MERGE

The `merge` statement is useful to upsert data - that is, update data when there is a matching record, or otherwise insert a new row.

It is split into the following components:

1. Merge into - specify the table you want to upsert the data
* Using - where you specify the query containing the source data you want to upsert
*  On - the match condition between the merge into and using data sources (typically, the primary key)
* When matched actions - you can an include and delete statements
* When not matched - an insert statement

Suppose we have the following two data sources set up - with employees_demo being the table we want to be the source of truth, and employees_extra as the updated data we'd like to merge.

```sql
create table employees_demo as
select *
from employees
where employee_id <= 105;
/

create table employees_extra as
select *
from employees
where employee_id <= 105;
/

update employees_extra
set salary = salary*2;
/

insert into employees_extra (employee_id, last_name, email, hire_date, job_id)
values (106, 'Carr', 'CARR', sysdate, 'AD_VP');

insert into employees_extra (employee_id, last_name, email, hire_date, job_id)
values (107, 'Perkins', 'PERKINS', sysdate, 'IT_PROG');
```

All of the insert, update, or delete statements can specify an additional where clause to apply a further filter - which can refer to either the source data or target table.

The `matched` clause would look like:

```sql
when matched
    then update
    set
        employees_demo.last_name = employees_extra.last_name,
        employees_demo.email = employees_extra.email,
        employees_demo.hire_date = employees_extra.hire_date,
        employees_demo.job_id = employees_extra.job_id,
        employees_demo.salary = employees_extra.salary
    where
        employees_extra.employee_id <= 110
    delete where employees_extra.employee_id  > 110 --No action here, just to demonstrate
```

And the `not matched` clause would like like:

```sql
when not matched
    then insert
        (
            employee_id,
            last_name,
            email,
            hire_date,
            job_id
        )
        values
        (
            employees_extra.employee_id,
            employees_extra.last_name,
            employees_extra.email,
            employees_extra.hire_date,
            employees_extra.job_id
        )
```

Which gives us an overall statement of:

```sql
merge into employees_demo
using (select * from employees_extra) employees_extra
on (employees_demo.employee_id = employees_extra.employee_id)
when matched
    then update
    set
        employees_demo.last_name = employees_extra.last_name,
        employees_demo.email = employees_extra.email,
        employees_demo.hire_date = employees_extra.hire_date,
        employees_demo.job_id = employees_extra.job_id,
        employees_demo.salary = employees_extra.salary
    where
        employees_extra.employee_id <= 110
    delete where employees_extra.employee_id > 110
when not matched
    then insert
        (
            employee_id,
            last_name,
            email,
            hire_date,
            job_id
        )
        values
        (
            employees_extra.employee_id,
            employees_extra.last_name,
            employees_extra.email,
            employees_extra.hire_date,
            employees_extra.job_id
        )
        where employees_extra.employee_id <= 110;
/
```
