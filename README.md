---
description: Welcome to the Stalactite documentation !
---

# Stalactite introduction

## What is it ?

Stalactite aims at being an ORM, but also gives some tools to ease JDBC usage : trying to be an ORM as a framework, but with some modules that helps to create CRUD SQL statements for Objects. To fulfill this goal, the project is devided in 3 main modules :

* The `orm` module : deals with relations between entities and allows persistence description through a fluent API. Depends on `core`.
* The `core` module : manages persistence of Objects without relation management. Depends on `sql`.
* The `sql`  module : a layer on top of JDBC to ease its usage.

One can use `core` or `sql` out of `orm`, but this documentation mainly talks about the `orm` module.

{% hint style="info" %}
Stalactite depends on 2 others projects called "Reflections" and "tools" under org.gama groupId, made to decouple things, but they are not intended to be used outside of this organisation : using them outside of it is not supported
{% endhint %}

## Why ? The genese

Stalactite is born after many years of Hibernate usage, with the observation that teams hardly master the framework even after years of practice :

* putting annotations on files is painless, but after some months of development, entity graph becomes a spaghetti plate where you can't get only a piece of it without loading the whole. Then you'll have to add additional annotation such as `@NamedEntityGraph`to specify what you want to load
* entity state (attached, detached, etc.) is still a problem for newcomers, and developpers still struggle with it after years of practise, or have rewritten (even if not optimized) a layer on top of repository to avoid to deal with it
* then come the well-known and common exceptions LazyInitializationException, StaleStateObjectException, ...

{% hint style="info" %}
**Some particular words about graph inflation**

By using annotations for mapping, we miss a non intrusive way of declaring it. HBM/XML is still possible but not officially well maintained. For instance, in a DDD project, where Domain must be framework-agnostic, annotations are a problem, then an abstraction layer can be introduced to decouple Domain Objects and Persistence Objects which is rarely more than a one-one mapping. As a second example, quickly persisting some data coming from a Controller is not possible before mapping them to entity which will produce boilerplate code or reflection overhead.

Finally, as long as the project rises in features, and produces relations between entities, the trend brings to obtain a huge entity graph if no one take care of it. Then performance for loading it may become a problem, as well as modifying it.
{% endhint %}

{% hint style="info" %}
This chapter mentioned Hibernate, but all of this can be applied to JPA in a more generic way.
{% endhint %}



## Disclaimer

In order to fulfill its goal (having an ORM less complex than Hibernate, with non intrusive mapping), some choices were made that may hardly change :

* **no query language** such as HQL/JPQL : because it's perfomance consuming while parsing the text, and because it's time consuming by its complexity for a developper which wants to implement such a project ! For now a fluent API is available for CRUD SQL statement writing, as well as a fluent API to describe the may of reading a produced ResultSet.
* **no attach/detach** concept, no first-level cache : because Stalactite doesn't need it at first. One reason for having the attach concept is to get an easy way to audit changes on entities, so the system can know which modifications should be sent to the database. Stalactite uses a difference algorithm computation to find those changes. This has a drawback : when updating an entity, the algorithm needs the modified version of the bean, as well as the unmodified version, usually coming from the database (but could be a serialized version of it, with risk of being out of sync with database of course). This implies that Stalactite will load your entity graph when you'll use the update(..) method, this may seems cumbersome but matches the load-update-persist pattern that happens behind the scene of Hibernate too.
* **no annotations, requires Java 8 at least** : as explained above, annotations are not the expected way to define mapping of entities because they are too intrusive. So the mapping will be done through Java Method References.
* **no lazy loading, promote bounded context / small aggregate roots** : in order to let developpers assess what they are doing with the database (in term of performance), and avoid lazy exceptions, **relations are always fetched**. By this way developper is encouraged to create several aggregates instead of one huge entity graph.

As a bonus Stalactite **doesn't use bytecode enhancement,** because it doesn't need to modify given beans nor their collections. But it uses reflection to apply values onto beans.

As a good pratice, only `PreparedStatement` are used to avoid SQL Injection. Meanwhile select statement are not totally protected because one can append any kind of Object in the SQL as parameter. So, as a developer, **you must care that SQL queries don't concatenate unprotected sources**.



{% hint style="danger" %}
### Known limitations :

* for now Stalactite supports only single column identifier
{% endhint %}



