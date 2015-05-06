# NCHR

`nchr` is very similar to the chr function, in that it returns the characterr value of the specified ascii code. One difference is that this functions return type is nvarchar2 - but the same could be achieved with [`chr`](chr.md) by specify `using nchar_cs`.

```sql
with chr_65 as (
select
    chr(65) chr1
  , chr(65 using nchar_cs) chr2
  , nchr(65) chr3  
from dual
)
select
    chr1
  , dump(chr1) dmp_chr1
  , chr2
  , dump(chr2) dmp_chr2
  , chr3
  , dump(chr3) dmp_chr3
from chr_65
```

Output:
```
CHR1 DMP_CHR1        CHR2 DMP_CHR2          CHR3 DMP_CHR3
---- --------------- ---- ----------------- ---- -----------------
A    Typ=1 Len=1: 65 A    Typ=1 Len=2: 0,65 A    Typ=1 Len=2: 0,65
```
