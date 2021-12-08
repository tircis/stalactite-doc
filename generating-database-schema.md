# Generating database schema

Stalactite provides a way to get your schema generated, which can be quite usefull in tests.

DDLDeployer aims at providing such a service : the simplest way is to give it the `PersistenceContext` used also for building your `Persister`s, because it will automatically discover `Table`s and `Column`s that were added to the context. Then we only have to invoke `deployDDL()` :&#x20;

```java
DataSource dataSource = ... // create one from whatever manner
PersistenceContext persistenceContect = new PersistenceContext(new JdbcConnectionProvider(dataSource), new MySQLDialect());

Persister<Country, Long, Table> countryPersister = MappingEase.entityBuilder(Country.class, Long.class)
    .add(Country::getId).identifier(IdentifierPolicy.afterInsert())
    .add(Country::getName)
    .build(persistenceContext);

DDLDeployer ddlDeployer = new DDLDeployer(persistenceContext);
ddlDeployer.deployDDL();
```



If you need, you can also add some tables by accessing its `DDLGenerator` :&#x20;

```
Table myTable = new Table("myTable);
myTable.addColumn("myColumn", String.class);

DDLDeployer ddlDeployer = new DDLDeployer(persistenceContext);
ddlDeployer.getDdlGenerator().addTables(myTable);
ddlDeployer.deployDDL();
```

