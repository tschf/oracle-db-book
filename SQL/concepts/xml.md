# XML Querying

The Oracle database has support for `XML` data types. You can verify that XML support is installed in your database by ensuring the `xdb` user exists, as well as doing a describe on `resource_view`.

```sql
select count(1)
from all_users
where username = 'XDB';
/

describe resource_view;
/
```

Output:
```
COUNT(1)
----------
       1

Name     Null Type
-------- ---- --------------
RES           XMLTYPE()
ANY_PATH      VARCHAR2(4000)
RESID         RAW(16 BYTE)
```

The type that denotes this field is `XMLType`. `XMLType` is unique in that it is both a data type and can be used as a constructor to return an `XMLType` data type. The first parameter can be CLOB or Varchar2, with optional parameters of: schema, validated, well formed. The constructor is a part of the [XMLType package](http://docs.oracle.com/cd/B19306_01/appdev.102/b14258/t_xml.htm#BABHCHHJ).

When calling the constructor, the `XML` that you pass in must be valid XML, otherwise you will receive an `ORA-31011: XML parsing failed` error.

`XMLTable` is a useful function that can be used used to extract particular bits of information out of the `XML`. It accepts an optional list of namespaces (used with the `XMLNamespaces` function), an XPath string that enables you to generate rows out of a single document (through recurring `XML` elements), and a list of columns that should be returned, where you again specify the XPath string to the relevant bit of information.

Assuming we have the following XML, stored in the table column xml_demo.xml_content

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

When defining the columns from within the `XMLTable` function, it is not strictly necessary to declare the columns data type - the database will make a best guess, as to what data type it should be (although, I always prefer to be explicit, in the data type I expect).

## Handling namespaces

The above works well right up until namespaces are introduced. Two possible scenarios here are that you have a default namespace, or an abbreviated namespace (that you can prefix elements with). So for a default namespace, if our XML changes to.

```xml
<?xml version="1.0"?>
<person id="1" xmlns="http://www.example.com/xmlperson">
    <name>Bob</name>
    <age>54</age>
    <location>Canada</location>
</person>
```

Re-running the above query will no longer work, as we haven't told the `XMLTable` function about our namespace. To solve this, we need to let the `XMLTable` function know about which namespace to reference. This is done by passing in the results of the `xmlnamespaces` function. For a default namespace as above, this is done by passing in the `DEFAULT` keyword and the quoted namespace URL.

```sql
select
    xml_doc.id
  , xml_doc.name
from
    xml_demo
  , xmltable (XMLNAMESPACES(DEFAULT 'http://www.example.com/xmlperson'),
        'for $i in //person
        return $i'
    passing xml_body
    columns
        name varchar2(50) path '//person/name',
        id NUMBER path '//person/@id'
    ) xml_doc
```

On the other hand, if you have prefixed namespaces and elements, in the declaration of the namespace, you would omit the `DEFAULT` keyword and alias the namespace to the same string as the prefix.

Also, when declaring the XPath string to specific elements, you also need to maintain the prefix in the path, otherwise the element's will not be found.

Suppose we have the following XML.

```xml
<?xml version="1.0"?>
<psn:person id="1" xmlns:psn="http://www.example.com/xmlperson">
    <psn:name>Bob</psn:name>
    <psn:age>54</psn:age>
    <psn:location>Canada</psn:location>
</psn:person>
```

Our query would become:

```sql
select
    xml_doc.id
  , xml_doc.name
from
    xml_demo
  , xmltable (XMLNAMESPACES('http://www.example.com/xmlperson' as "psn"),
        'for $i in //psn:person
        return $i'
    passing xml_body
    columns
        name varchar2(50) path '//psn:person/psn:name',
        id NUMBER path '//psn:person/@id'
    ) xml_doc
```

It's worth noting that you can still get the results to return without declaring the namespace - but in this case, you have to refer to the namespace with the wildcard character `*`, like so.

```sql
select
    xml_doc.id
  , xml_doc.name
from
    xml_demo
  , xmltable (
        'for $i in //*:person
        return $i'
    passing xml_body
    columns
        name varchar2(50) path '//*:person/*:name',
        id NUMBER path '//*:person/@id'
    ) xml_doc
```

Attempting to use the `psn` prefix will result in an error `ORA-19228: XPST0008 - undeclared identifier: prefix 'psn' local-name 'psn:person'`.
