---
description: Mapping a one-to-one relation
---

# One-to-one

Mapping a one-to-one relation can be done through the method `addOneToOne(..)` but it requires a mapping configuration for the target related entity.

```java
IFluentEntityMappingBuilder<City, Long> capitalMappingConfiguration = MappingEase.entityBuilder(City.class, Long.class)
    .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .add(City::getName);

MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId)
    .addOneToOne(Country::getCapital, capitalMappingConfiguration)
```

{% hint style="info" %}
Chaining those statements is encouraged in order to visualize graph complexity as relations are added : see [how to organize Stalactite persisters](../architecture-tips/dont-create-a-persister-to-rule-them-all.md). For instance :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .addOneToOne(Country::getCapital, MappingEase.entityBuilder(City.class, Long.class)
        .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
        .add(City::getName))
        .addOneToOne(City::getMayor, MappingEase.entityBuilder(Person.class, Long.class)
            .add(Person::getId).identifier(IdentifierPolicy.AFTER_INSERT)
            .add(Person::getFirstName)
            .add(Person::getLastName))
            .addOneToOne(Person::getCar, MappingEase.entityBuilder(Car.class, Long.class)
                .add(Car::getId).identifier(IdentifierPolicy.AFTER_INSERT)
                .add(Car::getModelName))
```
{% endhint %}

### Declaring property owned by reverse side

By default property relation is owned by declaring side, but depeding on your database design one may need to declare it on the target side, then method mappedBy(..) must be used :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .addOneToOne(Country::getCapital, MappingEase.entityBuilder(City.class, Long.class)
        .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
        .add(City::getName))
    .mappedBy(City::getCountry)
```

This creates a bidirectional one-to-one relation that will be persisted on a `City` table through the `countryId` column.

#### Without bidirectional relation

If you don't need bidirectionality, one may use the `mappedBy(..)` method with a `Column` parameter.

```java
Table cityTable = new Table("City");
Column<Table, Country> cityCountryPointer = city.addColumn("countryId", Country.class);
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .addOneToOne(Country::getCapital, MappingEase.entityBuilder(City.class, Long.class)
        .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
        .add(City::getName))
    .mappedBy(cityCountryPointer)
```

## Mandatory relation

As we did for a simple property, a one-to-one relation can be declared mandatory through the `madantory()` method :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .addOneToOne(Country::getCapital, MappingEase.entityBuilder(City.class, Long.class)
        .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
        .add(City::getName))
    .mandatory()
```

This implies that the owning property column is not nullable.

## Cascade type

By default, cascade policy is `ALL` (which means that unsaved target instance is save when root is saved, but not deleted when root is deleted), this behavior can be changed with the `cascading(..)` method :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .addOneToOne(Country::getCapital, MappingEase.entityBuilder(City.class, Long.class)
        .add(City::getId).identifier(IdentifierPolicy.AFTER_INSERT)
        .add(City::getName))
    .cascading(READ_ONLY)
```

{% hint style="info" %}
Please have a look to [cascading policies](cascading-policies.md) to choose the one that fits your needs.
{% endhint %}
