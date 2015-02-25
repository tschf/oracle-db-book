# BIN_TO_NUM

Oracle documentation: https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions020.htm#SQLRF00611

`bin_to_num` is used to convert a bit vector into a base 10 equivelant. Each parameter passed in represents a binary bit, so should only be a 1 or a 0.

```sql
select
    bin_to_num(0) zero,
    bin_to_num(1) one,
    bin_to_num(1,0) two,
    bin_to_num(1,1) three,
    bin_to_num(1,0,0) four,
    bin_to_num(1,0,1) five
from dual
```
Output:
```
      ZERO        ONE        TWO      THREE       FOUR       FIVE
---------- ---------- ---------- ---------- ---------- ----------
         0          1          2          3          4          5
```
