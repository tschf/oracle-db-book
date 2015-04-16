#INSTR

`instr` searches for one string, within another, and returns the starting index of the string being searched for. Another common name for this function in other languages is `indexOf`. If the search string is not found, `0` will be returned

There are five variants of this function:

1. instr - searches for chars
* instrb - searches for bytes
* instrc - searches using unicode complete chars
* instr2 - searches using unicode USC-4 code points
* instr4 - searches using unicode USC-4 code points

The function accepts at minimum two parameters, being the string you want to search and the string to search for. The other (two) optional parameters are the position to start from, and the occurance you would like to find.

```sql
with search_str as (
    select 'the quick brown fox jumped over the lazy dog' str
    from dual
)
select
    instr(str, 'the') search1
  , instr(str, 'the', 1) search2
  , instr(str, 'the', 1, 1) search3
  , instr(str, 'the', 1, 2) search4
  , instr(str, 'the', 2) searc5
  , instr(str, 'the', 34, 1) search6
from search_str
```
Output:
```
SEARCH1    SEARCH2    SEARCH3    SEARCH4     SEARC5    SEARCH6
---------- ---------- ---------- ---------- ---------- ----------
      1          1          1         33         33          0
```
