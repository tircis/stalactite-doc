---
description: Mapping a one-to-one relation
---

# One-to-many

Stalactite distinguishes `Set` from `List` to follow their semantic :&#x20;

* `Set` can't have duplicate, whereas `List` can
* `Set` are not indexed, `List` are. So, Stalactite will store index when `List` are used, not with `Set`.

{% hint style="warning" %}
Developers should more consider `Set`s when mapping collection, even if their application owner uses "List" when describing it's business, because most of the time it doesn't deal with duplicate nor index.
{% endhint %}

### Mapping Sets

When declaring your mapping, one can use `addOneToManySet` to map a Set.

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .add(Country::getName)
    .addOneToManySet(Country::getCities, MappingEase.entityBuilder(City.class, Long.class)
        .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
        .add(City::getName))
```

If your relation is owned by reverse side, then you'll find how to declare it below.

#### Sorting Sets

Since Stalactite doesn't use bytecode enhancement it leaves your `Set` untouched and won't instanciate one for you. This means that your field must be initialized before it is filled by Stalactite, else a NullPointerException will be thrown (initialization at construction time is a good place). This may look painfull but it leaves you a high degree of customization for it.

{% hint style="warning" %}
You may choose to use LinkedHashSet thinking that you'll keep entity collection adding order, but that's not true because you'll get `ResultSet` order which is not guaranteed to be steady accross SQL select execution : Databases don't guarantee this order, even if it is often (always ?) the same.
{% endhint %}

### Mapping Lists

Mapping a `List` is closed to mapping a Set : you'll have to use `addOneToManyList(..)` and use `indexedBy(..)` method to declare the index column, as the latter is optional Stalactite will use "idx" as a default name for this column.

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .add(Country::getName)
    .addOneToManyList(Country::getBiggestCities, MappingEase.entityBuilder(City.class, Long.class)
        .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
        .add(City::getName))
```

## Collections common mapping features

Hereafter are some common needs to `Set` and `List` mapping.

### Declaring property owned by reverse side

In both cases of `Set` and `List`, one may want to declare it as owned by reverse side. This can be done with same method as the one for one-to-one relation : `mappedBy(..)`

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .add(Country::getName)
    .addOneToManySet(Country::getCities, MappingEase.entityBuilder(City.class, Long.class)
        .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
        .add(City::getName))
    .mappedBy(City::getCountry)
```

#### Without bidirectional relation

If you don't need bidirectionality, one may use the `mappedBy(..)` method with a `Column` parameter.

```java
Table cityTable = new Table("City");
Column<Table, Country> cityCountryPointer = city.addColumn("countryId", Country.class);
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .add(Country::getName)
    .addOneToManySet(Country::getCities, MappingEase.entityBuilder(City.class, Long.class)
        .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
        .add(City::getName))
    .mappedBy(cityCountryPointer)
```

### Cascade type

By default, cascade policy is `ALL` (which means that unsaved target instance is save when root is saved, but not deleted when root is deleted), this behavior can be changed with the `cascading(..)` method :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .add(Country::getName)
    .addOneToManySet(Country::getCities, MappingEase.entityBuilder(City.class, Long.class)
        .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
        .add(City::getName))
	.cascading(READ_ONLY)
```

{% hint style="info" %}
Please have a look to [cascading policies](cascading-policies.md) to choose the one that fits your needs.
{% endhint %}
