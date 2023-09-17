# Migrating application data

This document is about migrating data when developing applications or the likes.
If you're not new to migrations, you may only want to read the section on
migrating JSON data (that is, creating migrations for such data). Otherwise,
let's start by defining what a migration is.

- [General concepts](#general-concepts)
- [Relational DB migrations (SQL)](#relational-database-migrations-sql)
- [Other migrations (JSON but not only)](#other-migrations-json-but-not-only)

## General concepts

What is a migration and how do we create one?

### Definition

A migration, which in our case stands for *data migration*, is a mechanism used
to make data change format (according to needs). For databases in particular, we
will rather talk about changing database structure. Basically, when data sources
that were valid in the past can no longer be processed, a migration should be
created to upgrade the data sources to the new expectations.

In practice, a migration can be carried out manually, but it is often preferable
to automate it, so that all the associated steps can be reproduced by executing
a single command from the CLI for example, given the exact data structure or a
copy of the data structure to be migrated; including that these steps could be
recorded alongside the source code using the same version control system.

### Examples

Data migration is not limited to any data format. Below are some examples using
object notations for clearer explanations.
- Rename the `name` field to `lastName`: `{name:'Arbitrary Name'}` &rarr; `{lastName:'Arbitrary Name'}`;
here, let's say a field is renamed because it is now referenced under a
different name that better reflects the underlying information.
- Convert `id` field to lowercase: `{id:'ABCDE6XYZTU'}` &rarr; `{id:'abcde6xyztu'}`;
a possible scenario would be that the IDs of many users need to be displayed in
lower case for a certain operation, so it's better to already have the IDs in
lower case rather than converting each one in the source code prior to display;
hence the migration.
- Replace all `firstName<number>` fields with a single `firstNames` field which
is an array: `{lastName:'Ln', firstName1:'Fn1', firstName2:'Fn2', ..., firstNameN:'FnN'}`
&rarr; `{lastName:'Ln', firstNames:['Fn1', 'Fn2', ..., 'FnN']}`; this is because
the new format is easier to handle.

### Implementation

At this stage, it should be clear that at least one operation is required when
migrating data: it's the `up()`, `apply()` or `whateverTheName()` function that
actually performs the migration. Another useful but optional operation is the `down()`,
`revert()` or `whateverTheName()` function to revert the migration when possible
after it has been executed. In Object-Oriented Programming, we could design an
abstract class or interface (`AbstractMigration`, `IMigration`, `Migration`, ...)
having one or more functions to override/implement in subclasses (`AddFieldXyz`,
`ConvertDataToNewFormatXyz`, `DeleteObsoleteUserAccounts`, ...). However,
regardless of the implementation, it's important to bear in mind that not all
migrations can be reverted: for example, once an entry has been deleted from a
database, it cannot be recreated unless its attributes are available somewhere
for later use.

Moreover, when it's necessary to create a system to manage all migrations,
such as detecting and running migrations that are not already executed, a new
class (`Migrator`, ...) could be introduced to find all available migrations,
check which ones are yet to be applied and execute them. In this case, the
migrations could be saved to the same location under timestamped file names (`migration_YYYY_MM_JJ_HH_MM_SS.*`,
`MigrationYYYYMMJJHHMMSS.*`, `YYYYMMJJHHMMSS_WhatTheMigrationDoes.*`, ...) so
that it's quick to find which migrations are to be executed based on the
timestamp in the name of the migration file that was executed last. For example,
if migration `2000/12/31 12:00:00` was executed last, then we know that
migration `2000/12/31 13:00:00` has not yet been executed, and of course
migration `2000/12/31 11:00:00` must not be created afterwards because it will
not be considered newer. Eventually, when it has been decided that the name of a
migration file would not reflect what the migration does, a `getDescription()`
function returning a brief description of a migration could be added to
migration classes.

## Relational database migrations (SQL)

Often, frameworks dedicated to Object Relational Mapping provide support for
database migrations. There are also tools specially designed to manage said
migrations. In these cases, the developer simply needs to create and run
migration files using the provided migration tools or process, so there's no
need to implement a migration system as described in the previous sections of
this document.

## Other migrations (JSON but not only)

### Context

Although data migration is a fairly popular concept for relational databases, at
least in the field of web development, sometimes the data to be migrated are
only available in a format that is not database-specific. For example, this
could be a viable choice for storing multi-layered data whose structure is not
rigid enough to be part of a static database schema, as illustrated below using
JSON-like syntax.

Storing application data received from external sources:

```js
// application data for applicant 1
{
    spanishLevel: 'B1-B2', // CEFR language level
}

// application data for applicant 2
{
    motivationText: '...',
    recommendedBy: {
        email: 'x@y',
        roleOrStatus: 'Physics teacher',
    },
}
```

### Example of migration in JavaScript

Now, how do we actually create migrations for JSON data or the likes? Here are a
few JavaScript functions to apply or revert a given migration.

```js
(function() {
    // applies the migration
    // obj reflects the data to be migrated, possibly read from file/others
    function up(obj) {
        // set a property on obj, assuming it doesn't exist (so no data loss)
        obj.x = obj.a + obj.b;

        // delete some properties from obj
        delete obj.a;
        delete obj.b;
    }

    // reverts the migration (which has already been applied)
    // obj reflects data that could have been obtained after up(), possibly read from file/others
    function down(obj) {
        const a = 1;
        obj.a = a;
        obj.b = obj.x - a; // see (1) below
        delete obj.x;

        // (1) assuming obj.a and obj.b are set to acceptable values, we can already see that their values before up()
        //     would only be restored by chance, because obj.a is arbitrarily set here, so down() should log an error
        //     message, raise an exception or not be implemented at all
    }
})();
```

### Design challenges when implementing a migration

Besides, if reading the data to be migrated involves mapping their sources to
the names of certain instance properties defined in a class, removing any of
these properties from the class may result in incorrect or impossible data
reading, for example when an XML parser reads XML data and uses a Reflection API
to introspect classes and automatically map the XML data to instances of those
classes, as illustrated below.

```xml
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<person>
    <bProp>true</bProp>
    <iProp>0</iProp>
    <sProp>...</sProp>
    <address>
        <iProp>0</iProp>
        <sProp>...</sProp>
    </address>
</person>

<!-- the following classes can be instantiated when the above XML data is read -->
<!--     class Person { bool bProp; int iProp; string sProp; Address addrProp; } -->
<!--     class Address { int iProp; string sProp; } -->
```

In this case, data classes might get a bit messy or complicated, unless there is
a library or framework that simplifies things for the developer, as properties
that are no longer used in business logic must be preserved in the source code
so that old data formats can be read correctly during migrations.

### Additional data migration methods

Finally, instead of creating runnable code for migrations as shown above, which
is probably the most transparent way to implement migrations, sometimes we might
prefer to rely on a third-party tool. For example, [jq](https://jqlang.github.io/jq/)
and [fx](https://github.com/antonmedv/fx) are possible options for manipulating
JSON data, as pointed out [here](https://stackoverflow.com/questions/70875281/automatically-migrate-json-data-to-newest-version-of-json-schema/70876163#70876163).
