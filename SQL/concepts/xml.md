# XML Querying

The Oracle database has support for `XML` data types. The type that denotes this field is `XMLType`. When you try and insert data into a column of this table it must by valid XML markup, otherwise you will receive an `ORA-31011: XML parsing failed` error.

You are then able to fetch attributes and values by using a the `XMLTable` function - which accepts the field containing the XML document, and then a series of XPath strings to define individual columns based on what is contained in the data.

Assuming we have the following XML, stored in the table column people.xml_content

```xml
<?xml version="1.0"?>
<person id="1">
    <name>Bob</name>
    <age>54</age>
    <location>Canada</location>
</person>
```

We could easily query this information like so:

```sql
select
    xml_doc.id
  , xml_doc.name
from
    xml_demo
  , xmltable (
        'for $i in //person
        return $i'
    passing xml_body
    columns
        name varchar2(50) path '/person/name',
        id NUMBER path '/person/@id'
    ) xml_doc
```

Giving us the following output:
```
ID          NAME
----------  -------------------------
1           Bob

```

When defining the columns from wihtin the xmltable function, it is not strictly necessary to declare the data type, and the database will make a best guess, as to what data type it should be.
