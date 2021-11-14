# Before insertion

This identifier policy will ask an id generator to give an identifier just before inserting a database row. The id generator is left open to user implementation, but a typical one is the `PooledHighLowSequence` which will store entity identifiers in a table (and one line per entity class), pooling them in memory to avoid too many back and forths with the database. Pool size is configurable and has 50 as default value. `PooledHighLowSequence`only generates `long` values.
