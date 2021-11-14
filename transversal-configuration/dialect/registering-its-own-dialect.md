# Registering its own Dialect

Dialects are discovered thanks to Java Service Provider system : Stalactite will look for subclassed of `DialectResolverEntry` in classpath. Hence some files named `org.gama.stalactite.persistence.sql.DialectResolver$DialectResolverEntry` must be present in classpath and contain implementation of `DialectResolverEntry`: this interface contains 2 methods working in couple :

* `getDialect()` is expected to return an instance of the Dialect to use
* `getCompatibility()` should return a `DatabaseSignet` that describes the database (name and version) with which the Dialect is compatible

Hence a subclass of `DialectResolverEntry` must be created for it : a simple way is to make your `Dialect` implements it (then `getDialect()` return this), or decouples it as it is done by default in Stalactite and create dedicated class entry.

