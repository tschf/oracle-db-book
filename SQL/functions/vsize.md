# VSIZE

`vsize` returns the number of bytes of an expression. Quite similar to [`lengthb`](LENGTH.md).

```sql
create table char_width(
    test_char varchar2(1 CHAR)
);
/

insert into char_width values ('Ä');

select test_char, lengthb(test_char), vsize(test_char)
from char_width
```

Output:

```
TEST_CHAR LENGTHB(TEST_CHAR) VSIZE(TEST_CHAR)
--------- ------------------ ----------------
Ä                          2                2
```
