# LENGTH

There are five variants of this function:

1. LENGTH - length in chars
* LENGTHB - length in bytes
* LENGTHC - length in unicode complete characters, meaning if it is a unicode complete character, it will return 1
* LENGTH2 - length using unicode USC-2 code points
* LENGTH4 - length using unicode USC-4 code points

```sql
create table char_width(
    test_char varchar2(1 CHAR)
);
/

insert into char_width values ('Ä');
insert into char_width values ('A');
/

select
    test_char
  , length(test_char) len
  , lengthb(test_char) lenb
  , lengthc(test_char) lenc
  , length2(test_char) len2
  , length4(test_char) len4
from char_width
```
Output:
```
TEST_CHAR        LEN       LENB       LENC       LEN2       LEN4
--------- ---------- ---------- ---------- ---------- ----------
Ä                  1          2          1          1          1
A                  1          1          1          1          1
```
If you don't specify the length semantics when creating the column, it'll most likely be stored in BYTE, verifiable with the following:

```sql
select parameter, value
from v$nls_parameters
where parameter = 'NLS_LENGTH_SEMANTICS';
```
Output:
```
PARAMETER               VALUE
----------------------- ---------
NLS_LENGTH_SEMANTICS    BYTE

```

Further reading:

* [List of unicode characters](http://en.wikipedia.org/wiki/List_of_Unicode_characters)
* [Unicode USC-2 Code chart](http://www.columbia.edu/kermit/ucs2.html)
* [Programming with unicode](http://unicodebook.readthedocs.org/en/latest/unicode_encodings.html)
