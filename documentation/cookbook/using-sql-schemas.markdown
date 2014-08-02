---
layout: configuration
title: Using SQL Schemas
---

# Using SQL Schemas #

Some database vendors support "schemas", a.k.a. namespaces of collections of database objects (tables, views, etc.).
MSSQL, PostgreSQL, and to a lesser extent MySQL, all provide the ability to group and organize tables into schemas.
Propel supports tables organized into schemas, and works seamlessly in this context. For SQLite we emulate schema support.

## Schema Definition ##

### Assigning a Table to a Schema ###

In a XML schema, you can assign all the tables included into a `<database>` tag to a given schema by setting the `schema` attribute on the `<database>` tag:

```xml
<database name="bookstore" schema="bookstore">
  <table name="book">
    <column name="id" required="true" primaryKey="true" autoIncrement="true" type="integer" />
    <column name="title" type="varchar" required="true" />
  </table>
</database>
```

>**Tip**On RDBMS that do not support SQL schemas (Oracle), the `schema` attribute is ignored.

You can also assign a table to a given schema individually ; this overrides the `schema` of the parent `<database>`:

```xml
<table name="book" schema="bookstore1">
  <column name="id" required="true" primaryKey="true" autoIncrement="true" type="integer" />
  <column name="title" type="varchar" required="true" />
</table>
```

### Foreign Keys Between Schemas ###

You can create foreign keys between tables assigned to different schemas, provided you set the `foreignSchema` attribute in the `<foreign-key>` tag.

```xml
<table name="book" schema="bookstore">
  <column name="id" required="true" primaryKey="true" autoIncrement="true" type="integer" />
  <column name="title" type="varchar" required="true" />
  <column name="author_id" type="integer" />
  <foreign-key foreignTable="author" foreignSchema="people" onDelete="setnull" onUpdate="cascade">
    <reference local="author_id" foreign="id" />
  </foreign-key>
</table>
<table name="author" schema="people">
  <column name="id" required="true" primaryKey="true" autoIncrement="true" type="integer" />
  <column name="name" type="varchar" required="true" />
</table>
```

## Schemas in Generated SQL ##

When generating the SQL for table creation, Propel correctly adds the schema prefix (example for MySQL):

```sql
CREATE TABLE `bookstore`.`book`
(
  `id` INTEGER NOT NULL AUTO_INCREMENT,
  `title` VARCHAR(255),
  PRIMARY KEY (`id`)
)
```

>**Tip**Propel does not take care of creating the schema. The target database must already contain the required schemas, and the user credentials must allow Propel to access this schema.

## Schemas in PHP Code ##

Just like actual table names, SQL schemas don't appear in the PHP code. For the PHP developer, who manipulates phpNames, it's as if schemas didn't existed.

Of course, you can make queries spanning across several schemas.

>**Tip**in Mysql, "SCHEMA" and "DATABASE" are synonyms. Therefore, the ability to define another schema for a given table actually allows cross-database queries.

## Using the Schema As Base for PHP code Organization ##

Propel provides other features to organize your model:

* _Packages_ are subdirectories in which Model classes get generated (see [Multi-Component Data Model](./multi-component-data-model.html))
* _Namespaces_ are actual PHP5.3 namespaces for generated Model classes (see [PHP 5.3 Namespaces](./namespaces.html))

You can easily tell Propel to copy the `schema` attribute to both the `package` and the `namespace` attributes, in order to reproduce the SQL organization at the PHP level. To that extent, modify the following settings in your *configuration file*:

<div class="conftabs">
<ul>
<li><a href="#tabyaml">propel.yaml</a></li>
<li><a href="#tabphp">propel.php</a></li>
<li><a href="#tabjson">propel.json</a></li>
<li><a href="#tabini">propel.ini</a></li>
<li><a href="#tabxml">propel.xml</a></li>
</ul>
<div id="tabyaml">
{% highlight yaml %}
propel:
  generator:
      schema:
          autoPackage: ture
          autoNamespace: true
{% endhighlight %}
</div>
<div id="tabphp">
{% highlight php %}
<?php

return [
    'propel' => [
        'generator' => [
            'schema' => [
                'autoPackage'   => true,
                'autoNamespace' => true,
            ]
        ]
    ]          
];
{% endhighlight %}
</div>
<div id="tabjson">
{% highlight json %}
{
    "propel": {
        "generator": {
            "autoPackage": true,
            "autoNamespace": true
        }
    }
}
{% endhighlight %}
</div>
<div id="tabini">
{% highlight ini %}
[propel]
;
; Generator section
; 
generator.schema.autoPackage = true
generator.schema.autoNamespace = true
{% endhighlight %}
</div>
<div id="tabxml">
{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <generator>
            <schema>
                <autoPackage>true</autoPackage>
                <autoNamespace>true</autoNamespace>
            </schema>
        </generator>
    </propel>
</config>
{% endhighlight %}
</div>
</div>


With such a configuration, a `book` table assigned to the `bookstore` schema will generate a `Bookstore\Book` ActiveRecord class under the `bookstore/` subdirectory.
