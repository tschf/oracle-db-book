# Packages

Every database session that calls a package, has its own instantiation of the package in memory. That means, when you assign values to variables at the package level, only that current session knows that state. That state will generally remain for the life of the session, excluding these 3 circumstances:

1. The package is re-compiled
* The package becomes invalidated
* The package is serially_reusable

So if we have the following.

```plsql
create table person_age (
    ID NUMBER PRIMARY KEY,
    NAME VARCHAR2(25) NOT NULL,
    AGE NUMBER NOT NULL
);
/

insert into person_Age values (1, 'Thomas', 54);
insert into person_Age values (2, 'Mary', 42);
insert into person_Age values (3, 'Peter', 77);
insert into person_Age values (4, 'Lynette', 12);
/
commit;
/

create or replace package person_Age_api
as

    procedure set_age(p_id in person_Age.id%type);

    function get_age(p_id in person_age.id%type)
    return NUMBER;

end person_age_api;
/

create or replace package body person_Age_api
as

    g_cached_age NUMBER := NULL;

    procedure set_age(p_id in person_Age.id%type)
    as
    begin

        if g_cached_age IS NULL
        then

            select age
            into g_Cached_age
            from person_age
            where id = p_id;
        else
            dbms_output.put_line('Age already set: ' || g_cached_age);
        end if;

    end set_age;

    function get_age(p_id in person_age.id%type)
    return NUMBER
    as
    begin

        return g_cached_age;

    end get_Age;

end person_age_api;
/

```

We can see the package losing its variable state.

```plsql
begin

    dbms_output.put_line('Age: ' || person_age_api.get_age(1));
    person_Age_api.set_Age(1);
    dbms_output.put_line('Age: ' || person_age_api.get_age(1));

end;
/

begin
    dbms_output.put_line('Age: ' || person_age_api.get_Age(1));
end;
/

create or replace package person_Age_api
as

    procedure set_age(p_id in person_Age.id%type);

    function get_age(p_id in person_age.id%type)
    return NUMBER;

end person_age_api;
/

begin
    dbms_output.put_line('Age: ' || person_age_api.get_Age(1));
end;
/
```

Output:
```
Age:
Age: 54

Age: 54

Package PERSON_AGE_API compiled

Age:
```
Which, you can see that re-compiling the package causes us to lose the state.

Then there is if the package is serially_reusable. Each time a package is instantiated in a database session, the state is stored in the user global area (UGA) of memory - which if there are lots of instantiations in different, can quickly fill up the UGA. On the other hand, a serially_reusable package stores use the system global area of memory - and doesn't persist for the life of a session - after the server process, any used memory is returned to the SGA.

In the specification of the package, you can specify the serially_reusable pragma. So, we can modify our earlier example to see this in action.

```plsql
create or replace package person_Age_api

as
    pragma serially_reusable;

    procedure set_age(p_id in person_Age.id%type);

    function get_age(p_id in person_age.id%type)
    return NUMBER;

end person_age_api;
/

create or replace package body person_Age_api
as
    pragma serially_reusable;
    g_cached_age NUMBER := NULL;

    procedure set_age(p_id in person_Age.id%type)
    as
    begin

        if g_cached_age IS NULL
        then

            select age
            into g_Cached_age
            from person_age
            where id = p_id;
        else
            dbms_output.put_line('Age already set: ' || g_cached_age);
        end if;

    end set_age;

    function get_age(p_id in person_age.id%type)
    return NUMBER
    as
    begin

        return g_cached_age;

    end get_Age;

end person_age_api;
/

begin

    dbms_output.put_line('Age: ' || person_age_api.get_age(1));
    person_Age_api.set_Age(1);
    dbms_output.put_line('Age: ' || person_age_api.get_age(1));

end;
/

begin
    dbms_output.put_line('Age: ' || person_age_api.get_Age(1));
end;
/
```

Output:

```
Age:
Age: 54

Age:
```
