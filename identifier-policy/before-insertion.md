# Before insertion

This identifier policy will ask an id generator to give an identifier just before inserting a database row. The id generator is left open to user implementation, but a typical one is the `PooledHighLowSequence` which will store entity identifiers in a table (and one line per entity class), pooling them in memory to avoid too many back and forths with the database. `PooledHighLowSequence`only generates `long` values.



Here is an example with it :

```java
// 50 is pool size, "mySequence" is sequence name, and default storage options are about table and column names that store the sequence
PooledHiLoSequence longSequence = new PooledHiLoSequence(new PooledHiLoSequenceOptions(50, "mySequence", SequenceStorageOptions.DEFAULT),
				persistenceContext.getDialect(),
				new TransactionalConnectionProvider(dataSource),
				persistenceContext.getJDBCBatchSize());


MappingEase.entityBuilder(Car.class, long.class)
    .add(Car::getId).identifier(IdentifierPolicy.beforeInsert(longSequence))
    .add(Car::getModel)
```
