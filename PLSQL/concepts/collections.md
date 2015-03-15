# Collections

A `collection` is a list of a elements. In other languages, this is typically thought of as an array or a list. Each element can be a simple data type, or a more complex user-defined type such as a table row, a record, or an object.

PL/SQL comes with three types of collections:

1. Associative arrays
* Varrays
* Nested table

## Associative arrays

An `associative array`, which depending on the language can also be referred to as a `map` or `dictionary`, is a list of elements which a lookup key. A key should either be a number or a string. Technically, the key can be other data types that can be converted to a string with `to_char`, however you should be caution here - if database parameters change, the returned string could also change and affect the results.

To create a variable that is an associative array, first you need to create a custom type, so that variables can be set to that specified data type with the general synax being: type typeName is table of _dataType_ index by _indexDataType_.

Associative arrays are strictly a PL/SQL data type, so cannot be created at the schema level as a `type`. You can declare them at the package level or block level (procedures, functions, anonymous blocks).

```plsql
declare
    type t_age_list is table of NUMBER index by varchar2(30);
    l_ages t_age_list;
begin
    --do stuff with l_ages;
end;
```

When you declare an associative array, it is automatically initialised with zero elements. In fact, it is not possible to assign NULL to the object at a whole, or check if it is NULL - you can only deal with individual element givens a particular `index`.


```plsql
declare
    type t_age_list is table of NUMBER index by varchar2(30);
    l_ages t_age_list;
begin

    if l_ages is null
    then
        dbms_output.put_line('Associative array is NULL');
    end if;
end;
```
Output:
```
Error report -
ORA-06550: line 6, column 8:
PLS-00306: wrong number or types of arguments in call to 'IS NULL'
ORA-06550: line 6, column 5:
PL/SQL: Statement ignored
06550. 00000 -  "line %s, column %s:\n%s"
*Cause:    Usually a PL/SQL compilation error.
*Action:
```
and
```plsql
declare
    type t_age_list is table of NUMBER index by varchar2(30);
    l_ages t_age_list;
begin
    l_ages := NULL;
end;
```
Output:
```
Error report -
ORA-06550: line 5, column 15:
PLS-00382: expression is of wrong type
ORA-06550: line 5, column 5:
PL/SQL: Statement ignored
06550. 00000 -  "line %s, column %s:\n%s"
*Cause:    Usually a PL/SQL compilation error.
*Action:
```

If you attempt to fetch a value with a key that doesn't yet have an element in the table, you will get the `no data found` error, much like you would trying to access data in a table.

```plsql
declare
    type t_age_list is table of NUMBER index by varchar2(30);
    l_ages t_age_list;
    l_toms_Age NUMBER;
begin
    l_toms_Age := l_ages('TOM');
    exception
        when no_data_found
        then
            dbms_output.put_line('No age for TOM yet');
end;
```

Output:
```
anonymous block completed
No element for TOM yet
```

### Iterating

Here, one technique is to make use of the `first` and `next` associative array functions. These functions will return the `key` of an element, where `first` will retrieve the first element of the array, and `next` will return the next key after the passed in parameter (key).

```plsql
declare
    type t_age_list is table of PLS_INTEGER
        index by varchar2(30);
    l_ages t_age_list;
    l_Age PLS_INTEGER;
    l_key varchar2(30);
begin
    l_ages('TOM') := 25;
    l_ages('PETER') := 55;
    l_ages('MARY') := 12;
    l_ages('MICHELLE') := 35;

    l_key := l_ages.FIRST;

    LOOP
        exit when l_key IS NULL;
        dbms_output.put_line(l_key || ' has an age of ' || l_ages(l_key));
        l_key := l_ages.NEXT(l_key);
    END LOOP;
    dbms_output.put_line('No more elements to read');
end;
```

Output:
```
anonymous block completed
MARY has an age of 12
MICHELLE has an age of 35
PETER has an age of 55
TOM has an age of 25
No more elements to read
```

## Varray

A `varray` is most similar to an array in other languages, where you have a fixed number of elements. One key difference though, most other languages allow you to pass in a variable containing the maximum number of elements whereas a varray requires the program to know the maximum number of elements without the use of a variable, so is less dynamic.

The general syntax is: type _typeName_ is varray(_size_) of _dataType_, and can be declared at the schema level or the block level.

```plsql
declare
    type t_four_strings is varray(4) of varchar2(10);
    l_names t_four_strings;
begin
    --do stuff with l_names
end;
```
or
```plsql
create or replace type t_four_strings is varray(4) of varchar2(10);
/

declare
    l_names t_four_strings;
begin
    --do stuff with l_names
end;
```

To use the varray, you need to initialise it. If I try to assign a value without first initialising it, I will get an exception.

```plsql
declare
    type t_four_strings is varray(4) of varchar2(10);
    l_names t_four_strings;
begin
   l_names(1) := 'TOM';
end;
```

Output:

```
Error report -
ORA-06531: Reference to uninitialized collection
ORA-06512: at line 5
06531. 00000 -  "Reference to uninitialized collection"
*Cause:    An element or member function of a nested table or varray
           was referenced (where an initialized collection is needed)
           without the collection having been initialized.
*Action:   Initialize the collection with an appropriate constructor
           or whole-object assignment.
```

When initialising the type, you can pass in the initial values. The number of values you pass in then becomes the current size of the `varray`. You can then increase it with the `extend` operation.

```plsql
declare
    type t_four_strings is varray(4) of varchar2(10);
    l_names t_four_strings;
begin
   l_names := t_four_strings();
   dbms_output.put_line('Varray size: ' || l_names.COUNT);
   l_names.extend();
   dbms_output.put_line('Varray size: ' || l_names.COUNT);
   l_names(1) := 'TOM';
   dbms_output.put_line('Element 1: ' || l_names(1));
   l_names.extend();
   l_names(2) := 'PETER';
   dbms_output.put_line('Element 2: ' || l_names(2));
   l_names.extend();
   l_names(3) := 'MARY';
   dbms_output.put_line('Element 3: ' || l_names(3));
   l_names.extend();
   l_names(4) := 'MICHELLE';
   dbms_output.put_line('Element 4: ' || l_names(4));
   dbms_output.put_line('All: ' || l_names(1) || ', ' ||l_names(2) || ', ' ||l_names(3) || ', ' ||l_names(4));
   --
   dbms_output.put_line('Re-initialise');
   l_names := t_four_strings('MICHELLE', 'PETER', 'MARY', 'TOM');
   dbms_output.put_line('Varray size: ' || l_names.COUNT);
   dbms_output.put_line('All: ' || l_names(1) || ', ' ||l_names(2) || ', ' ||l_names(3) || ', ' ||l_names(4));
end;
```

Output:

```
anonymous block completed
Varray size: 0
Varray size: 1
Element 1: TOM
Element 2: PETER
Element 3: MARY
Element 4: MICHELLE
All: TOM, PETER, MARY, MICHELLE

Varray size: 4
All: MICHELLE, PETER, MARY, TOM
```

`Varrays` can be set as the data type for a table column.

```plsql
create or replace type t_four_strings is varray(4) of varchar2(10);
/

create table name_list(
    ID NUMBER PRIMARY KEY
  , NAMES t_four_strings);
/

insert into name_list (ID, NAMES) values (1, t_four_strings('michelle', 'mary'));
insert into name_list (ID, NAMES) values (2, t_four_strings('peter', 'tom'));
insert into name_list (ID, NAMES) values (3, t_four_strings('peter', 'tom', 'frank'));
insert into name_list (ID, NAMES) values (4, t_four_strings('peter', 'tom', 'frank', 'michelle'));
/

select *
from name_list
```

Output:

```
	    ID NAMES
---------- ------------------------------------------------------------
	     1 T_FOUR_STRINGS('michelle', 'mary')
	     2 T_FOUR_STRINGS('peter', 'tom')
	     3 T_FOUR_STRINGS('peter', 'tom', 'frank')
	     4 T_FOUR_STRINGS('peter', 'tom', 'frank', 'michelle')

```

## Nested tables

A `nested table` is type of an array. It is different to a `varray` in that there is no upper limit and the index of elements doesn't necessarily go sequentially (in the case of removing elements). When you initialise a `nested array` the elements are accessed by a numerical subscript, and when values are added, they go into the nested array, in sequential order.

The syntax for creating the type is quite similar to an associative array, except you leave of the `index by` clause. Giving us the syntax: type _typeName_ is table of _dataType_. It can be created at the block level or the schema level, and can also be set as a columns data type in a table.

```plsql

declare
    type t_name_list is table of varchar2(10);
    l_names t_name_list;
begin
    --do stuff with l_names;
end;
```
or
```plsql
create or replace type t_name_list is table of varchar2(10);
/

declare
    l_names t_name_list;
begin
    --do stuff with l_names;
end;
```

On a nested table we can remove elements in the middle of the collection, which results in the subscript no longer following sequential order.

```plsql

declare
    type t_name_list is table of varchar2(10);
    l_names t_name_list := t_name_list('zxy', 'Two', 'Three', 'abc');
    l_element NUMBER;
begin

    dbms_output.put_line('List intialised nested table');
    l_element := l_names.FIRST;
    LOOP
        exit when l_element IS NULL;
        dbms_output.put_line(l_element || ': ' || l_names(l_element));
        l_element := l_names.NEXT(l_element);
    END LOOP;

    dbms_output.put_line('');
    dbms_output.put_line('After deleting 3rd element');
    l_names.delete(3);

    l_element := l_names.FIRST;
    LOOP
        exit when l_element IS NULL;
        dbms_output.put_line(l_element || ': ' || l_names(l_element));
        l_element := l_names.NEXT(l_element);
    END LOOP;

    dbms_output.put_line('');
    dbms_output.put_line('Extend the nested table');
end;


```

Output:
```
anonymous block completed
All elements
1: zxy
2: Two
3: Three
4: abc

Delete 3rd element
1: zxy
2: Two
4: abc
```

Nested tables can be set as the data type for a table column.

```plsql
create or replace type t_ages is table of NUMBER;
/

create table age_list(
    ID NUMBER PRIMARY KEY
  , AGES t_ages)
NESTED TABLE AGES store as nested_names;
/

insert into age_list (ID, AGES) values (1, t_ages(1,5,6,1));
insert into age_list (ID, AGES) values (2, t_ages(912,15,99));
insert into age_list (ID, AGES) values (3, t_ages(15,1,5,7));
insert into age_list (ID, AGES) values (4, t_ages(1));
/

select *
from age_list
```

Output:
```
        ID AGES
---------- ------------------------------
         1 T_AGES(1, 5, 6, 1)
         2 T_AGES(912, 15, 99)
         3 T_AGES(15, 1, 5, 7)
         4 T_AGES(1)
```
