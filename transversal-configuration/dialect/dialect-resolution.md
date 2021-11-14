# Dialect resolution

`ServiceLoaderDialectResolver` manages the way `Dialect` are found and choosen for a database connection. As mention is [Dialect registration paragraph](registering-its-own-dialect.md), Stalactite will use JVM service Provider to find available `Dialect`s, then it will select the "best matching" one according to database version (took from JDBC `Connection` metadata) and database compatibility given by `DialectResolverEntry` : vendor must match exactly and then the closest version is kept, with preference for smaller one. For example, with HSQLDB 2.5.2 (took from databse connection) an entry giving HSQLDB 2.5.0 will be prefered to a HSQLDB 3.0.0 one.

If this algorithm doesn't suit your need, one can change it by implementing its own `ServiceLoaderDialectResolver` and give it as argument to its `PersistenceContext` at instanciation time.
