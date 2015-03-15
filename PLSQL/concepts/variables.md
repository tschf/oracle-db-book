# Variables

Variables are names properties that store some sort of value. In PL/SQL they can be any of the SQL data types, as well as any custom data types that have been declared. The PL/SQL language also has the `BOOLEAN` data type that is not supported in SQL, which supports the values `TRUE`, `FALSE` or `NULL`.

The general syntax is: _variable_name_ _data_type_;

```plsql
declare

    l_str varchar2(10);
    l_num NUMBER;
    l_clb CLOB;
    l_bool BOOLEAN;
    --etc

begin

    l_str := 'str';
    l_num := 1;
    l_clb := 'large str';
    l_bool := TRUE;

    dbms_output.put_line(l_Str);
    dbms_output.put_line(l_num);
    dbms_output.put_line(l_clb);
    dbms_output.put_line(case when l_bool then 'TRUE' else 'FALSE' end);

end;
```
Output:
```
anonymous block completed
str
1
large str
FALSE
```

## Inherit data type

There is a special attribute `%type` that can inherit attributes of column tables and other variable declarations. At least for the case of using this with table columns, if you column attributes change, your program will automatically inherit these updated properties.

To use this attribute, you simple append it to the object that you want to inherit the properties of.

```plsql
create table inherit_demo(
    str varchar2(10)
);
/

declare
    l_str inherit_demo.str%type;
    l_str2 l_str%type;
begin
    l_Str := 'test';
    l_Str2 := 'test2';
end;
/
```

## Initialisation

When you declare a variable, you can also give it an initial value. Do so by either using the assignment operator `:=` or specifying `default` _val_, as per the create table syntax

```plsql
declare
    l_Str varchar2(10) := 'str';
    l_str2 varchar2(10) default 'str2';
begin
    dbms_output.put_line(l_Str);
    dbms_output.put_line(l_Str2);
end;
```

As a best practice, I tend to avoid giving initial values in the declare blocks, apart from the case of `constants` (discussed down the page). Not only does it tend to be the last place you'd think the error to occur in, declare blocks don't support exception handlers.

Consider that we can catch any value errors with the following:

```plsql
declare
    l_str varchar2(10);
begin

    l_str := 'a string more than 10 chars';
    dbms_output.put_line(l_str);

    exception when value_error
    then
      dbms_output.put_line('l_str assignment too large');
      --raise;
end;
```
Output:
```
anonymous block completed
l_str assignment too large
```

However, move the assignment into the declare block and we aren't afforded the same benefit of landing in the exception block:

```plsql
declare

    l_str varchar2(10) := 'a string more than 10 chars';

begin

    dbms_output.put_line(l_str);

    exception when value_error
    then
      dbms_output.put_line('l_str assignment too large');
      --raise;
end;
```

Output:
```
Error report -
ORA-06502: PL/SQL: numeric or value error: character string buffer too small
ORA-06512: at line 3
06502. 00000 -  "PL/SQL: numeric or value error%s"
*Cause:    An arithmetic, numeric, string, conversion, or constraint error
           occurred. For example, this error occurs if an attempt is made to
           assign the value NULL to a variable declared NOT NULL, or if an
           attempt is made to assign an integer larger than 99 to a variable
           declared NUMBER(2).
*Action:   Change the data, how it is manipulated, or how it is declared so
           that values do not violate constraints.
```

## Constants

If you have a variable you don't want the value to change, you can declare it with the `constant` modifier, which is placed before the data type and after the variable name. Although the previous section discussed not setting variable values in the declare block, we don't really have an option here - so just be aware of the points discussed when declaring constants.

```plsql
declare
    lc_id CONSTANT NUMBER := 1;
begin
    dbms_output.put_line(lc_id);
end;
```

## Not NULL

As per columns in a table, variables can also be forced to have a value with the `not null` constraint specification, which you specify after the data type - much like you would in a `create table` statement.

In this case, the variable must have an `initialisation` value. If not, you will receive the following error:

```
Error report -
ORA-06550: line 2, column 11:
PLS-00218: a variable declared NOT NULL must have an initialization assignment
06550. 00000 -  "line %s, column %s:\n%s"
*Cause:    Usually a PL/SQL compilation error.
*Action:
```

Add to that, if you try and assign NULL in your programs body, you will receive the following error:

```plsql
declare
    l_Str varchar2(10) NOT NULL := 'str1';
    l_str2 varchar2(10) default 'str2';
begin
    dbms_output.put_line(l_Str);
    l_str := NULL;
    dbms_output.put_line(l_Str2);
end;
```
Output:
```
Error report -
ORA-06550: line 6, column 14:
PLS-00382: expression is of wrong type
ORA-06550: line 6, column 5:
PL/SQL: Statement ignored
06550. 00000 -  "line %s, column %s:\n%s"
*Cause:    Usually a PL/SQL compilation error.
*Action:
```

If the data type is inherited from a table column that has a NOT NULL constraint, the constraint isn't inherited along with it, so you would need to specify NOT NULL again.

## Scope

Variables have block level scope in PL/SQL. When you nest blocks within one another, the PL/SQL engine first looks for the variable in the current scope. If it can't find that variable, it will look at the parent block, and so on.

```plsql
declare
    a_name varchar2(10);
    b_name varchar2(10);
begin

    a_name := 'ab';
    b_name := 'zy';

    declare
        a_name varchar2(10);
    begin
        dbms_output.put_line(nvl(a_name,'-'));
        dbms_output.put_line(b_name);
        a_name := 'cd';
        dbms_output.put_line(a_name);
    end;

end;
```
Output:
```
anonymous block completed
-
zy
cd
```
Which as you can see, in the inner block `a_name` is looking up the variable from that block, but where the block doesn't have a scope for a variable name, it will look at the next level up.

You can still refer to variables in the parent block. In the case of an anonymous block, you would need to give it a label, using the label identifiers `<<name>>`. Then the variable from that block can be accessed with _label.variable_.

```plsql
<<super>>
declare
    a_name varchar2(10);
    b_name varchar2(10);
begin

    a_name := 'ab';
    b_name := 'zy';

    <<sub>>  
    declare
        a_name varchar2(10);
    begin
        dbms_output.put_line(super.a_name);
        dbms_output.put_line(b_name);
        a_name := 'cd';
        dbms_output.put_line(sub.a_name);--equivelant to dbms_output.put_line(a_name);
        super.a_name := 'zz';
        dbms_output.put_line(a_name);
    end;
end;
```
Output:
```
anonymous block completed
ab
zy
cd
zz
```

If you are using a compiled procedure, you can access variables with _procedureName.variable_.

```plsql
create or replace procedure var_Scope
as
    a_name varchar2(10);
begin
    a_name := 'ab';
    dbms_output.put_line(a_name);
    <<var_scope>>
    declare
        a_name varchar2(10);
    begin
        dbms_output.put_line(var_Scope.a_name);
        a_name := 'zy';
        var_scope.a_name := 'ac';
        dbms_output.put_line(a_name);
    end;

    dbms_output.put_line(a_name);
end var_Scope;
/

begin

    var_scope;

end;
/
```
Output:
```
anonymous block completed
ab
ab
zy
ac
```

If you have a procedure name and a block label with the same name, the closest one would be used.

```
create or replace procedure var_Scope
as
    a_name varchar2(10);
begin
    a_name := 'ab';
    dbms_output.put_line(a_name);
    <<var_scope>>
    declare
        a_name varchar2(10);
    begin
        dbms_output.put_line(nvl(var_Scope.a_name,'-'));
        a_name := 'zy';
        var_scope.a_name := 'ac';
        dbms_output.put_line(a_name);
    end;

    dbms_output.put_line(a_name);
end var_Scope;
/

begin

    var_scope;

end;
/
```
or similarly:
```plsql
<<var_scope>>
declare
    a_name varchar2(10);

    procedure var_scope
    as
        a_name varchar2(10);
    begin
        dbms_output.put_line(nvl(var_scope.a_name, '-'));
        var_scope.a_name := 'ac';
        dbms_output.put_line(a_name);
    end var_scope;
begin

    a_name := 'ab';
    dbms_output.put_line(a_name);
    var_scope;
    dbms_output.put_line(a_name);
end;
```

Output:

```
anonymous block completed
ab
-
ac
ab
```
