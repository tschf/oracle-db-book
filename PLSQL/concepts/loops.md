# Loops

The most basic kind of loop begins with the keyword `loop` and ends with the keyword `end loop`.

```plsql
begin
    LOOP
        --dostuff
    END LOOP;
end;
```

The above example will run indefinitely, so you either need to provide an `exit` statement in the loop body, or use a for loop that controls when the loop will end.

## Exit

The above loop has no exit condition, so will run forever. Here we can add an `exit` clause to the loop like so:

```plsql
declare
    counter NUMBER := 1;
begin
    LOOP
        dbms_output.put_line(counter);
        if counter = 10
        then
            exit;
        end if;

        counter := counter+1;
    END LOOP;
end;
```

However, a more concise way to call `exit` is with a `when` condition.

```plsql
declare
    counter NUMBER := 1;
begin
    LOOP
        dbms_output.put_line(counter);

        exit when counter = 10;

        counter := counter+1;
    END LOOP;
end;
```
Output:
```
anonymous block completed
1
2
3
4
5
6
7
8
9
10
```

Naturally, the exit statements relates to the closest loop that the statement lives in. It may be the case you would like to exit the outer loop. To do that, you can make use of labels and specify which loop you would like to exit out of.

```plsql
declare
    outer_counter NUMBER := 1;
    inner_counter NUMBER;
begin
    <<main_loop>>
    LOOP
        dbms_output.put_line('Outer: ' || outer_counter);
        inner_counter := 1;
        <<nested_loop>>
        LOOP
            dbms_output.put_line('Inner: ' || inner_counter);
            exit main_loop when outer_counter = 2 and inner_counter = 2;
            exit when inner_counter = 3;
            inner_counter := inner_counter + 1;
        END LOOP;

        exit when outer_counter = 3;
        outer_counter := outer_counter + 1;
    END LOOP;
end;
```
Output:
```
anonymous block completed
Outer: 1
Inner: 1
Inner: 2
Inner: 3
Outer: 2
Inner: 1
Inner: 2
```

## Range for loops

Here, we can use the range operator `..` to loop over a series of numbers, where the syntax is _startNum..endNum_.

Typically like `for i in 1..10`. On each iteration, i is a variable that you can use for the life of the loop that gets assigned the next value in the range.

```plsql
declare
    max_count NUMBER := 3;
begin
    for i in 1..max_count
    loop
        dbms_output.put_line(i);
    end loop;
end;
```

Output:
```
anonymous block completed
1
2
3
```

## While loop

The other type of loop construct that you can use is a `while` loop. The `while` statement follows some condition that is evaluated on each iteration.

```plsql
declare
    counter NUMBER := 1;
begin
    while counter <= 3
    loop
        dbms_output.put_line(counter);
        counter := counter + 1;
    end loop;
end;
```
Output:
```
anonymous block completed
1
2
3
```

## Continue

The `continue` statement can be used to stop execution of the loop and start the next iteration. Much like the `exit` statement, `continue` can be on it's own, or used in combination with the `when` statement.

```plsql
declare
    counter NUMBER := 1;
begin
    LOOP
        dbms_output.put_line('Starting ' || counter);

        counter := counter + 1;
        exit when counter > 5;
        if counter = 3
        then
            continue;
        end if;

        dbms_output.put_line('Finishing iter');

    END LOOP;
end;
```
or
```plsql
declare
    counter NUMBER := 1;
begin
    LOOP
        dbms_output.put_line('Starting ' || counter);
        counter := counter + 1;

        exit when counter > 5;
        continue when counter = 3;

        dbms_output.put_line('Finishing iter');
    END LOOP;
end;
```
Output:
```
anonymous block completed
Starting 1
Finishing iter
Starting 2
Starting 3
Finishing iter
Starting 4
Finishing iter
Starting 5
```
