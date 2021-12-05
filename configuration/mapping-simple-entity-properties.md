---
description: First steps to map an Entity
---

# Mapping simple Entity properties

From the `orm` module, the entry point is `MappingEase`. The first step is to give class entity and its identifier type to the `entityBuilder(..)` method, such as this :

```java
MappingEase.entityBuilder(Country.class, Long.class)
```

Then you can add simple properties by chaining them with the `add(..)` method

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId)
    .add(Country::getName)
```

{% hint style="info" %}
Note that setters can be used as well :
{% endhint %}

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::setId)
    .add(Country::setName)
```

## Identifier

After the `add(..)` method, one can use the `identifier(..)` method to declare the property as the identifying one. An identifier management policy must be given as argument.

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .add(Country::getName)
```

In a next chapter we'll discuss about identifier policy.

## Additional options

### Declaring property as mandatory

If you want Stalactite to check against property nullity you can call the `mandatory()` method after mapping a property

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.ALREADY_ASSIGNED)
    .add(Country::getName).mandatory()
```

This will add a non null constraint on the database column, and will throw an exception in case of null value at insertion and update time.

{% hint style="success" %}
Please note that **primitive type properties are considered mandatory**, so it is not necessary to declare them as mandatory
{% endhint %}

### Setting a column name

You can override a property column name by adding it as an argument of the `add(..)` method.&#x20;

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.ALREADY_ASSIGNED)
    .add(Country::getName, "isoCode")
```

### Setting a column

If you already have a `Table` instance (a Stalactite `orm` class) representing the database table and its `Column` instances you can reference them in the `add(..)` method, as you do for a column name.

Column type and property type must match, this is guaranteed by generics type of the `add(..)` method signature.

```java
// Declaring a Table representing the one of the database schema
Table countryTable = new Table("country");
Column<Table, String> isoCodeColumn = countryTable.addColumn("isoCode", String.class);

MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.ALREADY_ASSIGNED)
    .add(Country::getName, isoCodeColumn)
```
