# CV

`cv` is used when [modelling](../concepts/MODELLING.md) your query. It is generally used in the rules portion of the model for getting the relative value of the matched row, given the dimension column passed in.

E.g. increment all salaries:

```sql
select
    employee_id,
    first_name,
    salary
from employees
model
    dimension by (employee_id)
    measures (first_name, salary)
    rules (
        salary[employee_id] = salary[cv(employee_id)] * 1.1
    )
```
