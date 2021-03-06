Like other DBMS's, Postgres has a built-in `xml` data type. The advantage of 
using this type (as opposed to storing the data as text) is that it can 
validate for well-formed XML. There are some built-in functions as well.

### Store XML data

Let's create a new table:

```
CREATE TABLE xmltest (
    id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
    xmlcol xml
);
```{{execute}}

Let's attempt to add this XML, which is missing the closing tag `</events>`:

```
INSERT INTO xmltest (xmlcol) VALUES ('
<events xmlns="https://example.com">
    <event>
        <name>Individual appointment</name>
        <date>05-25-2020</date>
        <mode>Virtual</mode>
    </event>
    <event>
        <name>Resume and cover letter workshop</name>
        <date>05-31-2020</date>
        <mode>In-person</mode>
    </event>
');
```{{execute}}

You should get back:
 >ERROR: invalid XML content

Let's try again with valid XML:

```
INSERT INTO xmltest (xmlcol) VALUES ('
    <events xmlns="https://example.com">
        <event>
            <name>Individual appointment</name>
            <date>05-25-2020</date>
            <mode>Virtual</mode>
        </event>
        <event>
            <name>Resume and cover letter workshop</name>
            <date>05-31-2020</date>
            <mode>In-person</mode>
        </event>
    </events>
');

SELECT * FROM xmltest;
```{{execute}}

### Processing XML: `xpath` function

The [`xpath` function](https://www.postgresql.org/docs/current/functions-xml.html#FUNCTIONS-XML-PROCESSING)
 can be used to process XML data. Here's an example that returns the `name` 
 values from the XML using a path expression:

```
SELECT xpath('//mydefns:name/text()', 
        xmlcol,
        ARRAY[ARRAY['mydefns', 'https://example.com']])
FROM xmltest;
```{{execute}}

>**Note:** 
>
>It is not possible to directly index an `xml` column. 
>
>XML support in Postgres was added mainly for compliance, and options for 
processing XML are still relatively limited.

### Links to official documentation

[postgresql.org: XML Type](https://www.postgresql.org/docs/current/datatype-xml.html)  
[postgresql.org: XML Functions](https://www.postgresql.org/docs/current/functions-xml.html)