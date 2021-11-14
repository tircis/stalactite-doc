---
description: A quick introduction about using Stalactite orm module
---

# Persistence configuration

The entry point to use Stalactite from the `orm `module is the `MappingEase.entityBuilder(..)` method which helps one to build a `Persister` : it is the object that let one insert, update, delete or select an entity and its graph.

To build a `Persister`, a `PersistenceContext` is necessary : it contains database oriented informations such as a JDBC`Connection` provider and a `Dialect`.

Here is an example:

```java
DataSource dataSource = ... // create one from whatever manner
PersistenceContext persistenceContect = new PersistenceContext(new JdbcConnectionProvider(dataSource), new MySQLDialect());

Persister<Country, Long, Table> countryPersister = MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId)
    .add(Country::getName)
    // ... configuration is explain in next chapters
    .build(persistenceContext);
    
Country country = new Country(1);
country.setName("France");

countryPersister.persist(country);
countryPersister.delete(country);

Country loadedCountry = countryPersister.select(1);
```
