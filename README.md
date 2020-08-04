---
description: Welcome to the Stalactite documentation !
---

# Stalactite introduction

## What is it ?

Stalactite aims at being an "ORM Library" : trying to be an ORM as a framework but with some modules that let one uses it as an Object-Persistence library, or SQL layer. To fulfill this goal, the project is devided in 3 modules :

* The `orm` module : aims at managing relations between entities and mapping persistence with a fluent API. Depends on `core`.
* The `core` module : manages persistence of classes without relation management. Depends on `sql`.
* The `sql`  module : a layer on top of JDBC to ease its usage \(in the context of Stalactite\).

One can use `core` or `sql` out of `orm`, but this documentation talks about the `orm` module.

\(Stalactite depends on 2 others projects called "Reflections" and "tools" under org.gama groupId, made to decouple things, but they are not intended to be used outside of this organisation : using them outside of it is not supported\)

## Why ? The genese

Stalactite is born after many years of Hibernate usage, with the observation that teams hardly master the framework even after years of practice :

* putting annotations on files is painless, but after some months of development, entity graph becomes a spaghetti plate where you can't get only a piece of it without loading the whole.
* entity state \(attached, detached, etc.\) is still a problem for new comers as well as some developpers that ends by doing always the same things \(even if not optimized\) without knowing what they are doing
* then come the well-known exceptions LazyInitializationException, StaleStateObjectException, ...

{% hint style="info" %}
**Some particular words about graph inflation**

By using annotations for mapping, we miss a non intrusive way of declaring it. HBM/XML is still possible but not officially well maintained. For instance, in a DDD project, where Domain must be framework-agnostic, annotations are a problem, then an abstraction layer is introduced for decoupling Domain Objects and Persistence Objects which is rarely more than a one-one mapping. As a second example, persisting DTO is not possible before mapping them to entity which will produce boilerplate code or reflection overhead.

Finally, as long as the project rises in features, producing relationship between entities, the trend brings you to obtain a huge entity graph if no one take care of it. Then performance for loading it may become a problem, as well as modifying it.
{% endhint %}

{% hint style="info" %}
This chapter mentioned Hibernate, but all of this can be applied to JPA
{% endhint %}



## Disclaimer

In order to fulfill its goal \(having an ORM less complex than Hibernate, with non intrusive mapping\), some choices were made that may hardly change :

* **no query language** such as HQL/JPQL : because it's perfomance consuming while parsing the text, and because it's time consuming by its complexity for a developper which wants to start such a project ! For now a SQL fluent API is available as well as a fluent API to describe the may of mapping a produced ResultSet.
* **no attach/detach** concept, no first-level cache : because Stalactite doesn't need it. One reason for having the attach concept is to get an easy way to audit changes on entities, so the system can know which modifications should be sent to the database. Stalactite uses a difference algorithm computation to find those changes. This has a drawback : when updating an entity, you have to give the modified version of the bean, as well as the unmodified version \(coming from the database for instance\). This enforces ORM arcana understanding to the developper but looks cumbersome. A future enhancement may be done to simplify the update inputs.
* **no annotations, requires Java 8 at least** : as explained above, annotations are not the expected way to define mapping of entities because they are too intrusive. So the mapping will be done throught Java 8 Method References.
* **no lazy loading, promote bounded context / small aggregate roots** : in order to let developpers assess what they are doing with the database \(in term of performance\), and avoid lazy exceptions, **relations are always fetched**. By this way developper is encouraged to create several aggregates instead of one huge entity graph.

As a bonus Stalactite **doesn't use bytecode enhancement,** because it doesn't need to modify given beans nor their collections. But it uses reflection to apply values onto beans.

As a good pratice, only PreparedStatement are used to avoid SQL Injection, at least on write statement. Meanwhile select statement are not totally protected because one can append any kind of Object in the SQL as parameter. So, as a developer, **you must care that SQL queries don't concatenate unprotected sources**.



{% hint style="danger" %}
### Known limitations :

* for now Stalactite supports only single column identifier
{% endhint %}





