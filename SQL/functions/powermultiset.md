# POWERMULTISET

`powermultiset` returns a nested table of nested tables. It returns a nested table of all possible subset combinations of nested tables. Best demonstrated by creating a nested table containing three values to see the return nested table.

```sql
create or replace type locations_ot is object (
    location_ud number(4,0),
    street_address varchar2(40),
    postal_code varchar2(12),
    city varchar2(30),
    state_province varchar2(25),
    country_id char(2)
);
/

create or replace type locations_ntt is table of locations_ot;
/

create or replace TYPE locations_nntt
  AS TABLE OF locations_ntt;
/

create table employees_demo as
select * from employees;
/

ALTER TABLE employees_demo
ADD (locations_list locations_ntt)
NESTED TABLE locations_list STORE AS locations_list_ntt_store;
/

update employees_demo
set locations_list =
    cast(
        multiset(
            select
                location_id,
                street_Address,
                postal_code,
                city,
                state_province,
                country_id
            from locations
            where location_id in (1000, 1100) )
        as locations_ntt
    )
where employee_id = 100;
/

select cast(powermultiset(locations_list) as locations_nntt) power_multi
from employees_demo
where employee_id = 100
```
Output:
```
POWER_MULTI
-----------
HR.LOCATIONS_NNTT(
  HR.LOCATIONS_NTT(
    HR.LOCATIONS_OT(1000,'1297 Via Cola di Rie','00989','Roma',NULL,'IT')
  )
, HR.LOCATIONS_NTT(
    HR.LOCATIONS_OT(1100,'93091 Calle della Testa','10934','Venice',NULL,'IT')
  )
, HR.LOCATIONS_NTT(
    HR.LOCATIONS_OT(1000,'1297 Via Cola di Rie','00989','Roma',NULL,'IT')
  , HR.LOCATIONS_OT(1100,'93091 Calle della Testa','10934','Venice',NULL,'IT')
  )
, HR.LOCATIONS_NTT(
    HR.LOCATIONS_OT(1200,'2017 Shinjuku-ku','1689','Tokyo','Tokyo Prefecture','JP')
  )
, HR.LOCATIONS_NTT(
    HR.LOCATIONS_OT(1000,'1297 Via Cola di Rie','00989','Roma',NULL,'IT')
  , HR.LOCATIONS_OT(1200,'2017 Shinjuku-ku','1689','Tokyo','Tokyo Prefecture','JP')
  )
, HR.LOCATIONS_NTT(
    HR.LOCATIONS_OT(1100,'93091 Calle della Testa','10934','Venice',NULL,'IT')
  , HR.LOCATIONS_OT(1200,'2017 Shinjuku-ku','1689','Tokyo','Tokyo Prefecture','JP')
)
, HR.LOCATIONS_NTT(
    HR.LOCATIONS_OT(1000,'1297 Via Cola di Rie','00989','Roma',NULL,'IT')
  , HR.LOCATIONS_OT(1100,'93091 Calle della Testa','10934','Venice',NULL,'IT')
  , HR.LOCATIONS_OT(1200,'2017 Shinjuku-ku','1689','Tokyo','Tokyo Prefecture','JP')
  )
)
```
