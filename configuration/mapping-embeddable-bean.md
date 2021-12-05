---
description: How to embed a bean onto an Entity
---

# Mapping embeddable bean

Embedding beans follows the same principle as properties or enums, the method to use on a configuration is `embed(..)`.

Supposing that a `Country` has a `Timestamp` which itself as a `modificationDate` and a `creationDate` that you want to persist into the same database table than the `Country` entity, then you have to embed the `Timestamp` (complex type) into the `Country` entity in such the following way :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .embed(Country::getTimestamp)
```

### Overriding properties column names

Sometimes embedded bean property names don't match column ones, this requires to indicate column name that the property must fall into. This can be done by using the method `overrideName(..)` after declaring the embedded property :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .embed(Country::getTimestamp)
        .overrideName(Timestamp::getCreationDate, "creation_date")
```

### Excluding properties from embedding

By default all target entity properties (please be aware of the note above : word "properties" may be misused) are candidates for being persisted, this may not be wanted and some properties should be removed from persistence. For such case one could use the `exclude(..)` method :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .embed(Country::getTimestamp)
        .exclude(Timestamp::getCreationDate)
```

{% hint style="warning" %}
Be aware that candidates are computed from getter and setter methods (following Java Bean Naming convention), not from instance fields.
{% endhint %}

### Embedding of embedding

In some cases one may need to embed a bean of an already-embedded bean, then you should use the `innerEmbed(..)` method as in this (totally fake) exemple where president modification date is used as the election date stored in the country table :

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .embed(Country::getPresident)
        .innerEmbed(Person::getTimestamp)
            .exclude(Timestamp::getCreationDate)
                .overrideName(Timestamp::getModificationDate, "presidentElectedAt")
```
