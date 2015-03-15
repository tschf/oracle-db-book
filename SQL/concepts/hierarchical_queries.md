# HIERARCHICAL QUERIES

Otherwise knows as `tree queries`, are recursive queries that refer to the same entity, through some key field.

The syntax is to specify the `connect by` clause, followed by the condition. Typically, you would also add the `prior` operator immediately before the condition, which causes the database to look at the parent row(s) in the hierarchy.

Then, you would specify the `start with` clause to identify the root row. Without this clause, the result set will include each manager as the root node.

```sql
select employees.employee_id, employees.first_name, employees.last_name, level
from employees
where employee_id = 101
connect by prior employee_id = manager_id  start with manager_id is NULL;


select employees.employee_id, employees.first_name, employees.last_name, level
from employees
where manager_id = 101
connect by prior employee_id = manager_id  start with manager_id is NULL;
```
Output:
```
EMPLOYEE_ID FIRST_NAME           LAST_NAME                      LEVEL
----------- -------------------- ------------------------- ----------
        101 Neena                Kochhar                            2


EMPLOYEE_ID FIRST_NAME           LAST_NAME                      LEVEL
----------- -------------------- ------------------------- ----------
        108 Nancy                Greenberg                          3
        200 Jennifer             Whalen                             3
        203 Susan                Mavris                             3
        204 Hermann              Baer                               3
        205 Shelley              Higgins                            3
```

Compare that to if we leave off the `start with` clause:

```sql
select employees.employee_id, employees.first_name, employees.last_name, level
from employees
where employee_id = 101
connect by prior employee_id = manager_id ;


select employees.employee_id, employees.first_name, employees.last_name, employees.manager_id, level
from employees
where manager_id = 101
connect by prior employee_id = manager_id ;
```
Output:
```
EMPLOYEE_ID FIRST_NAME           LAST_NAME                      LEVEL
----------- -------------------- ------------------------- ----------
        101 Neena                Kochhar                            1
        101 Neena                Kochhar                            2

EMPLOYEE_ID FIRST_NAME           LAST_NAME                 MANAGER_ID      LEVEL
----------- -------------------- ------------------------- ---------- ----------
        108 Nancy                Greenberg                        101          2
        205 Shelley              Higgins                          101          2
        204 Hermann              Baer                             101          2
        203 Susan                Mavris                           101          2
        200 Jennifer             Whalen                           101          2
        108 Nancy                Greenberg                        101          1
        205 Shelley              Higgins                          101          1
        204 Hermann              Baer                             101          1
        203 Susan                Mavris                           101          1
        200 Jennifer             Whalen                           101          1
        108 Nancy                Greenberg                        101          3
        205 Shelley              Higgins                          101          3
        204 Hermann              Baer                             101          3
        203 Susan                Mavris                           101          3
        200 Jennifer             Whalen                           101          3

 15 rows selected
```

Which as you can see, that particular supervisor is then reported at each supervisor level, and the children are listed at each child level.

As is demonstrable in the above examples, the where clause is evaluated after running the connect by operation.

Of the returned rows, the root row is then identified, and subsequently any child rows, which helps to set the pseudocolumn `level` which identifies how many levels deep a particular row is, in a query.

You do need to be cautious of loops that may exist. For instance, if we have Neena's manager as Nancy and Nancy's manager as Neena, we have each person refering back to each other, so when leaving off the `start with` clause we would have an infinite loop.

```sql
create table employees_demo as select * from employees;
/

update employees_demo set manager_id = 108 where employee_id = 101;
/

select employees.employee_id, employees.first_name, employees.last_name, employees.manager_id, level
from employees_demo employees
where employee_id = 101
connect by prior employee_id = manager_id
```
Output:
```
Error report -
SQL Error: ORA-01436: CONNECT BY loop in user data
01436. 00000 -  "CONNECT BY loop in user data"
*Cause:
*Action:
```

The `nocycle` clause can help to avoid this.

```sql
select employees.employee_id, employees.first_name, employees.last_name, employees.manager_id, level
from employees_demo employees
where employee_id = 101
connect by nocycle prior employee_id = manager_id
```
Output:
```
EMPLOYEE_ID FIRST_NAME           LAST_NAME                 MANAGER_ID      LEVEL
----------- -------------------- ------------------------- ---------- ----------
        101 Neena                Kochhar                          108          2
        101 Neena                Kochhar                          108          1
```
