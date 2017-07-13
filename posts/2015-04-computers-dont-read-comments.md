This is a draft specification for a self-defining database schema using a row-oriented database, such as MySQL. I reserve the right to update this document at any time :)

This post is basically a brain dump to try to nail down the naming convention which should be used in databases to make the names easily parseable and readable in a processing environment, with as little configuration, and as little operational limitation as possible. I plan to update this document as I use this specification, as I fully expect it to evolve over time.

<h2>table/entity naming convention</h2>

<ul>
<li>Table names must match exactly the class object names in our database. All entity names are suffixed with 'Entity', as are our table names.</li>
<li>Table names must be less than 32 chars for readability, but are limited to 64 chars in length, including the length of `Entity` - so bear this in mind when naming your entities.</li>
<li>Table names must begin with a capital letter, not a number, and contain only the following letters `[A-Za-z0-9]`. </li>
<li>Special characters and spaces are prohibited.</li>
</ul>

<pre>
abcdefghijklmnopqrstuvwxyzabcdef
#                              ^ 32 chars (quite long)         v 64 chars (silly)
abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijkl
</pre>

Example valid table names:

`FooEntity`, `Bar123Entity`, `ALongerNameEntity`, `ANameWithAnAcronymABCDEntity`

We use a common suffix on our entity names and tables for the following reasons: 
<ul>
<li>to easily map our tables to entities</li>
<li>to easily map our entities to tables</li>
<li>to prevent using reserved names in PHP</li>
<li>to prevent using reserved names in MySQL</li>
<li>to easily identify when we are working with an Entity in our application</li>
</ul>

<h2>field/property naming convention</h2>

Field names must contain all the description required to properly submit or read the data from the database, or to determine the proper relationship with another table.

`&lt;prefix&gt;&lt;CamelCaseName&gt;`

prefix:
`e` = enum (always stored as an int in the database, never use database `enum` type)
`s` = string (varchar, longtext, shorttext, mediumtext, etc.)
`i` = int (tinyint, smallint, mediumint, int, bigint, etc.)
`f` = float (float, double, single, etc.)
`b` = bool (`tinyint(1)` = `1` or `0`)
`o` = object (stored as an int in the database, contains ID of the foreign key)
`r` = raw binary (usually a blob)
`d` = datetime (should be used for date, datetime, time, and timestamp types)
`x` = internal, reserved, used for mapping

Prefixes are compulsory - the length/format is never checked, assumed the database will accept any length output by the system, but should fail gracefully when exceeded.
Case sensitive columns names - the CamelCaseName is just a name, has no processed meaning.

One exception to this case - the `id` field is always present in every table, and is used to uniquely reference each row. This is also recorded as the `id` field in the entity class.

If the entity is referred to by other entities and you need this relationship in reverse to be automatically populated into an array, then the object type needs to be an `array` within the entity definition, and an `int` within the table definition, also following the `object` naming convention above.

<pre>
<?php
public class OrderEntity
{
    const XDBMAP_OPERSON = 'PersonEntity';

    /** @var PersonEntity */
    public $oPerson;
}
public class PersonEntity
{
    const XDBMAP_OHOUSEHOLDENTITY = 'HouseholdEntity';
    const XDBMAP_OMANAGER = 'PersonEntity';
    const XDBMAP_OSTAFF = '[PersonEntity.oManager]';

    /** @var HouseholdEntity */
    public $oHousehold;

    /** @var PersonEntity */
    public $oManager;

    /** @var PersonEntity */
    public $oStaff = [];
}
public class HouseholdEntity
{
    const XDBMAP_OOCCUPANTS = '[PersonEntity.oHousehold]';

    /** @var PersonEntity[] */
    public $oOccupants = [];
}
</pre>
<h2>enums</h2>

ENUMS within the database are prohibited. They unnecessarily complicate matters, especially when it comes to modifying the enum values.

Indexed lookups, referential integrity, etc. may be used - the application has no awareness of them.

Entities within the database that are defined from a CONST value held in the Entity class definition should be defined with the prefix `i`, `s`, `f` or `b` as shown above, and not linked to a foreign table. The principle behind this is that list entities in the database should only be present when adding to or removing from the list affects the application without having to modify the code for the change to take effect. This also stands up to database normalisation rules - the data should not be duplicated, i.e. it should not be stored as a CONST value AND be defined in the database.
<pre>
<?php
public class FooEntity
{
    const
        ESERVICETYPE_FOO = 1,
        ESERVICETYPE_BAR = 2,
        ESERVICETYPE_FAR = 3,

    const
        ESERVICETYPE__1 = 'I am Foo',
        ESERVICETYPE__2 = 'We are Bar',
        ESERVICETYPE__3 = 'You are Far',

    /** @var int */
    public $id;

    /** @var int */
    public $eServiceType = self::ISERVICETYPE_FOO;
}
</pre>
The example above describes a field in the database being represented as an integer - there is no functionality to be gained from having the field name stored in a separate table in the database, if modifying any of the 3 values above would have no effect without a code change.
Note the naming convention here - the const names above exactly match the variable name, in uppercase only. The underscore separates the name from the machine value, and the const is defined in a block to indicate to the developer that these options apply to the same field.
Note the additional const names - these names are for display purposes only, and describe the machine value stored in the field. So the internal value of FooEntity::FOO is 1, and the human-readable version of that is 'I am Foo', used for display purposes only, and never stored - this value might be subject to change, based on your application.
If more complex enums are required, such as the storage of string-value enums, then a separate table and object class should be used instead. The above configuration is intended to suit most use cases, but will not suit all.

Consts with a double-underscore separating the name from the value are used for display purposes, e.g. `ESERVICETYPE__3` is a const field that contains the display value for `ESERVICETYPE_FAR`, which has a const value of 3. This enables easy lookup of the display value given the stored value. Now you are starting to see the importance of this field always being numeric.

Furthermore, the const display fields are not compulsory, but their existence will not be checked prior to their usage - so if you <b>do</b> use the fields for display purposes, make sure you have the const values properly configured. EAFP - It is easier to ask for forgiveness than permission.

<h2>nulls</h2>
Whether any field in the table can be stored as `null` is entirely down to you - the application will not care, and will fail with a handled error if a null value is attempted to be stored in a non-null field.

<h2>relationships</h2>
If the field type is an object (prefixed with `o`), then the field name should be suffixed with the name of the database table to which it refers, separated with an underscore.

<h2>default values</h2>
Default values should be specified in the entity class as defaults for the class properties, in the example above `self::ISERVICETYPE_FOO` is the default value for `$eServiceType`.
If default values are required that are not able to be specified in the parameters, like `new \DateTime()`, then these should be set in the entity `__construct()` method.
