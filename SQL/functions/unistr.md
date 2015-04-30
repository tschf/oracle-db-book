#UNISTR

`unistr` allows you to convert a unicode hex number into the specified character, by proceeding the hex number after the backlash character.

If you look at this currency table: http://www.unicode.org/charts/PDF/U20A0.pdf , you can find the different unicode hex numbers associated, and to demonstrate a couple from the table:

```sql
select UNISTR('\20A4') gbp, unistr('\20AC') euro
from dual
```

Output:
```
GBP EURO
--- ----
₤   €
```
