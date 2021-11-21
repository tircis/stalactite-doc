---
description: How to build a bean graph from some SQL
---

# Complex projection

{% hint style="info" %}
Following explanations will use example of a query created through simple `String`s, please note that all methods mentionned below also exist with a signature accepting `Query` object which can be build thanks to a fluent API starting with `QueryEase.selec(..)`.
{% endhint %}

## **From an SQL statement**

An object can be created from a simple SQL query thanks to the `PersistenceContext#newQuery(CharSequence, Class)` method. The first argument contains the SQL to be executed, and the second is the type of bean built by the execution of the query. Then one can map selected columns to object properties through method references :\


```java
persistenceContext.newQuery("select name, isoCode from Country", Country.class)
    .map("name", Country::setName)
    .map("isoCode", Country::setIsoCode)
    .execute();
```

By default the property type will be used to read `ResultSet` column, but if the column type doesn’t really match the bean property one then we can enforce it by giving the type to use as an extra `map(..)` method parameter such as below. This will tell Stalactite to use the matching-type `ResultSetReader` configured at the `Dialect` level.

```java
persistenceContext.newQuery("select name, inhabitantCount, isoCode from Country", Country.class)
    .map("name", Country::setName)
    .map("inhabitantCount", Country::setInhabitantCount, long.class)
    .map("isoCode", Country::setIsoCode)
    .execute();
```

At last, if some conversion needs to be done, you can specify it with some extra arguments to the `map(..)` method : the first extra one is the Java type for the column to be read, and the second extra one is the converter. It will get the column value as input and its result will be given to the method :

```java
persistenceContext.newQuery("select name, nuclearWeaponCount, isoCode from Country", Country.class)
    .map("name", Country::setName)
    .map("nuclearWeaponCount", Country::setHasNuclearWeapon, int.class, weaponCount -> weaponCount > 0)
    .map("isoCode", Country::setIsoCode)
    .execute();
```

## **Setting request parameter**

Of course one can give parameters to the query and set them through `set(..)` methods. Parameters are named (not indexed) and must be prefixed by “:” in SQL, such as below :

```java
persistenceContext.newQuery("select name, isoCode from Country where isoCode = :code", Country.class)
    .set("code", "FR")
    .singleResult()
    .execute();
```

{% hint style="warning" %}
Note that Stalactite will detect parameters and transform SQL as a `PreparedStatement` to avoid SQL injection. Meanwhile parameter detection is a kind of “simple and stupid” algorithm and won’t take all cases into account for now.
{% endhint %}

## **Creating a bean graph**

Previous paragraphs create one bean per row and a very flat object but Stalactite allows the creation of complex tree by defining object keys and relations. For instance one can create an entity by using `mapKey(..)` methods (there are several signatures of them) to define the identifier of root bean. In example below `Country` class has a one-arg constructor that gets the instance identifier :

```java
persistenceContext.newQuery("select id, name, isoCode from Country", Country.class)
    .mapKey(Country::new , "id", long.class)
    .map("name", Country::setName)
    .map("isoCode", Country::setIsoCode)
    .execute();
```

Stalactite also supports multiple-args constructor, supposing that `Country` has a 3-args constructor you can write :

```java
persistenceContext.newQuery("select id, name, isoCode from Country", Country.class)
    .mapKey(Country::new , "id", long.class, “name”, String.class, “isoCode”, String.class)
    .execute();
```

Furthermore, since first argument of `map(..)` methods is a `Function`, factory method pattern is also supported: supposing that `Country` only exposes a static factory method called “fromDatabase” and not its private constructors, we can use it as well :

```java
persistenceContext.newQuery("select id, name, isoCode from Country", Country.class)
    .mapKey(Country::fromDatabase, "id", long.class, “name”, String.class, “isoCode”, String.class)
    .execute();
```

{% hint style="success" %}
Actually, thanks to the latter case, any factory method can be used, whether it is exposed by the entity class itself or not.
{% endhint %}

{% hint style="warning" %}
When multiple constructors are present in entity class, due to method reference detection by Java, you may have to cast given constructor reference as the right one else a compilation error will be raised :
{% endhint %}

```java
persistenceContext.newQuery("select id, name, isoCode from Country", Country.class)
    .mapKey((SerializableFunction<Long, Country>) Country::new , "id", long.class)
    .map("name", Country::setName)
    .map("isoCode", Country::setIsoCode)
    .execute();
```

## Using a table metamodel

As an architecture strategy, a project may decide to expose its Database model as a metamodel, this has several benefits if you write it with Stalactite `Table`s and` Column`s, in particular a query can be mapped to bean properties by using `Column`s. Supposing the project defines a `CountryTable` class which is a `Table` class with accessible `Column` as instance field. One can write following code :

```java
CountryTable countryTable = new CountryTable();
persistenceContext.newQuery("select id, name, isoCode from Country", Country.class)
    .mapKey(Country::new , countryTable.id)
    .map(countryTable.name, Country::setName)
    .map(countryTable.isoCode, Country::setIsoCode)
    .execute();
```

This way of writing ensures that your bean properties match column types since method signatures are made for it.

Moreover you can combine it with fluent API select clause writing to obtain a very refactoring-robust query :

```java
CountryTable countryTable = new CountryTable();
persistenceContext.newQuery(QueryEase.select(countryTable.id, countryTable.name, countryTable.isoCode).from(countryTable), Country.class)
    .mapKey(Country::new , countryTable.id)
    .map(countryTable.name, Country::setName)
    .map(countryTable.isoCode, Country::setIsoCode)
    .execute();
```

## Returning one row

For very simple cases one can expect the request to return only one row, therefore the query should not return a `List` but a single `Object` or `null`. To get such a behavior, just call `singleResult()` during mapping :

```java
int countryCount = persistenceContext.newQuery("select count(*) as countryCount from Country", Country.class)
    .map(Function.identity(), “countryCount”, int.class)
    .singleResult()
    .execute();
```

## Building a complex model

The query API lets also build a graph by attaching related beans to the root one through its accessor methods and a `ResultSetRowTransformer` that will manage related bean construction. Note that accessor methods will be abstracted through a BeanRelationFixer, so one can benefit from its available static methods to, for example, manage bidirectionality.

### To-one relation

```java
persistenceContext.newQuery("select Country.id as countryId, Country.name as countryName, Country.isoCode, City.id as cityId, City.name as cityName from Country inner join City on Country.capitalId = City.id", Country.class)
    .mapKey(Country::new , "id", long.class)
    .map("countryName", Country::setName)
    .map("isoCode", Country::setIsoCode)
    .map(BeanRelationFixer.of(Country::setCapital), new ResultSetRowTransformer<>(City.class, “cityId”, DefaultResultSetReaders.LONG_PRIMITIVE_READER, City::new)
    .execute();
```

### To-many relation

Attaching a collection to a root bean will use same method as to-one relation : since its first argument is a combiner of 2 beans we must create a smart one that will check collection presence and add related bean, whereas passed `ResultSetRowTransformer` remains a bean creator as mentioned for to-one relation.

```java
// Note that code below is equivalent to BeanRelationFixer.of(Country::setCities, Country::getCities, HashSet::new)
BeanRelationFixer<Country, City> cityCountryCombiner = (country, city) -> {
    if (country.getCities() == null) {
	country.setCities(new HashSet<>());
    }
    country.getCities().add(city);
};


persistenceContext.newQuery("select Country.id as countryId, Country.name as countryName, Country.isoCode, City.id as cityId, City.name as cityName from Country inner join City on Country.capitalId = City.id", Country.class)
    .mapKey(Country::new , "id", long.class)
    .map("countryName", Country::setName)
    .map("isoCode", Country::setIsoCode)
    .map(cityCountryCombiner, new ResultSetRowTransformer<>(City.class, “cityId”, DefaultResultSetReaders.LONG_PRIMITIVE_READER, City::new)
    .execute();
```

## Very open mapping

If you don’t find the appropriate method to build your result, one may finally consume the read `ResultSet` thanks to a very open `map(..)` method signature that gets a consumer of root bean and `ResultSet` :

```java
persistenceContext.newQuery("select Country.id as countryId, Country.name as countryName, Country.isoCode, City.id as cityId, City.name as cityName from Country inner join City on Country.capitalId = City.id", Country.class)
    .mapKey(Country::new , "id", long.class)
    .map("countryName", Country::setName)
    .map("isoCode", Country::setIsoCode)
    .map((rootBean, resultSet) -> {
            City city = new City(resultSet.getLong(“cityId”));
            city.setName(resultSet.getString(“cityName”));
            rootBean.setCity(city);
        })
    .execute();
```

\




