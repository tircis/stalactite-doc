---
description: First steps to map an Entity
---

# Mapping simple Entity properties

From the `orm` module, the entry point is `MappingEase`. The first step is to give class entity and its identifier type to the `mappingBuilder(..)` method, such as this :

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

At any time after the `add(..)` method, one can use the `identifier(..)` method to declare the property as the identifying one. An identifier management policy must be given as argument.

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.ALREADY_ASSIGNED)
    .add(Country::getName)
```

{% hint style="warning" %}
#### **About already-assigned identifier strategy**

Because this strategy means that before behind inserted entities get an identifier \(such as an UUID, a business key, but nothing that requires any database roundtrip such as any sequence-like mecanism\), it doesn't let Stalactite distinguish persisted Entities from non persisted ones \(since both have identifiers\), preventing to choose the right SQL action of insert or update. To solve it, the `StatefullIdentifier` interface was introduced : ****it acts as a wrapper around any identifier value and adds a "persisted" state to it, then Stalactite can manage it and applies the good SQL operation. 2 concrete classes are provided to satisfy this interface :

* PersistableIdentifier which must be used when instanciating new Entities,
* PersistedIdentifier which must be used to discribe an already persisted Entity

**As a result, for now Entities must implement the Stalactite `Identified` interface which implies that they must have a `getId()` method which returns a Stalactite `Identifier` object. As a consequence, selection and deletion by id should be made with an identifier such as `select(new PersistedIdentifier<>(1L)),` not by identifier value.**

**Some enhancement around this will take place "soon" since this emplies model dependency on Stalactite.**
{% endhint %}

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
Please note that **primitive type properties are considered mandatory**, so it is not required to declare them as mandatory
{% endhint %}

### Setting a column name

You can override a property column name by adding it as an argument of the `add(..)` method. 

```java
MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.ALREADY_ASSIGNED)
    .add(Country::getName, "isoCode")
```

### Setting a column

If you already have a `Table` instance \(of the Stalactite `orm` API\) representing the database table and its `Column` instances you can reference them in the `add(..)` method, as you did for a column name.

Column type and property type must match, this is guaranteed by generics type of the `add(..)` method signature.

```java
// Declaring a Table representing the one of the database schema
Table countryTable = new Table("country");
Column<Table, String> isoCodeColumn = countryTable.addColumn("isoCode", String.class);

MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.ALREADY_ASSIGNED)
    .add(Country::getName, isoCodeColumn)
```

