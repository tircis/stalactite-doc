# Mapping enum

Mapping enum looks very close to simple property mapping, the thing that changes is calling the `addEnum(..)` method instead of the `add(..)` method

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.afterInsert())
    .addEnum(Country::getLanguage)
```

{% hint style="info" %}
By default, enum names are used as persistence value, not their ordinal position. Hereafter you'll learn how to change this behavior.
{% endhint %}

### Using ordinal position as persistence value

if you need to use enum ordinal values you can specify it with the `byOrdinal()` method after the `addEnum(..)` method :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.afterInsert())
    .addEnum(Country::getLanguage).byOrdinal()
```

### Ensuring enum name as persistence value

As said before, **enum names are used by default so the following is not necessary**, but one may need to specify it. This can be done in the same way as for ordinal value : call `byName()` after the `addEnum(..)` method :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.afterInsert())
    .addEnum(Country::getLanguage).byName()
```

## Additional options

### Declaring property as mandatory

If you want Stalactite to check against property nullity you can call the `mandatory()` method after mapping the enum

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.afterInsert())
    .addEnum(Country::getLanguage).mandatory()
```

This will add a non null constraint on the database column, and will throw an exception in case of null value at insertion and update time.

### Setting a column name

You can override the enum column name by adding it as an argument of the `addEnum(..)` method.&#x20;

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.afterInsert())
    .addEnum(Country::getLanguage, "locale")
```

### Setting a column

If you already have a `Table` instance (a Stalactite `orm` class) representing the database table and its `Column` instances you can reference them in the `addEnum(..)` method, as you did for a column name.

Column type and property type must match, this is guaranteed by generics type of the `addEnum(..)` method signature.

```java
// Declaring a Table representing the one of the database schema
Table countryTable = new Table("country");
Column<Table, String> isoLocaleColumn = countryTable.addColumn("locale", Locale.class);

MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.afterInsert())
    .addEnum(Country::getLanguage, isoLocaleColumn)
```
