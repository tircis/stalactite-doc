# Selecting entities from criteria

Beyond classic possibility to select an entity through its id, Stalactite provides a way to retrieve them through some criteria that match their properties : criteria are targetted through their method reference as they were defined during mapping.

{% hint style="warning" %}
Only properties used by mapping can be used as criteria. If a criteria contains a non-mapped property then an exception will be raised.
{% endhint %}



Here is a simple example to get all countries whose name starts with "F", operators are available in class `org.gama.stalactite.query.model.Operators`.

```java
EntityPersister<Country, Long> persister = entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(afterInsert())
    .add(Country::getName)
    .build(persistenceContext);
    
List<Country> countries = persister.selectWhere(Country::getName, startsWith("F"))
        .execute();
```



It also worlk with relation, herafter with a one-ton-many :

```java
RelationalEntityPersister<Country, Long> persister = (RelationalEntityPersister) entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(afterInsert())
    .add(Country::getName)
    .addOneToManySet(Country::getCities, entityBuilder(City.class, Identifier.class)
        .add(City::getId).identifier(afterInsert())
        .add(City::getName))
        .mappedBy(City::getCountry)
    .build(persistenceContext);
    
List<Country> select = persister.selectWhere(Country::getName, startsWith("F"))
        .andMany(Country::getCities, City::getName, eq("Grenoble"))
	.execute();
```

Beware that result returned by `entityBuilder(..).build(..)` must be casted to `RelationalEntityPersister` to allow chaining criteria for relations such as `andMany(..)`.
