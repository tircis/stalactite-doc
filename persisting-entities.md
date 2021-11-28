# Persisting entities

Building your mapping through `MappingEase.entityBuilder(..).build(persistenceContext)` let's you get an `EntityPersister` which then allows one to apply usual commands on your entities :&#x20;

* `EntityPersister.insert(entity)` will insert given entity with an insert order
* `EntityPersister.update(entity)` will update given entity with an update order after having loaded it from database in order to compute differences between it and given one. It takes optimistic locking into account
* `EntityPersister.persist(entity)` will insert or update given entity according to its persistence state, which is based on entity id and depends on id policy. Please note that in case of necessary update, entity will be loaded from database since it is delegated to `update(entity)`
* `EntityPersister.delete(entity)` will delete given entity with a delete order. It takes optimistic locking into account
* `EntityPersister.select(id)` will load a whole entity graph with a select order, or several of them if graph is too complex to be loaded through a single select with join (polymorphism may be a good reason)

{% hint style="info" %}
All of these methods have a massive variant signature accepting an `Iterable`.
{% endhint %}



Above were the basic commands, but more advanced ones exist :

* `EntityPersister.update(entity, allColumnStatement)` allows you to decide weither or not all columns will be contained in SQL order, if not (second argument set to`false`), only modified columns will be present in SQL. Be aware that using it as such may have performance impact since JDBC batch will have poor change to be activated while updating multiple entities, unless their modifications target same properties. This option is applied to all entity graph.
* `EntityPersister.update(id, entityConsumer)` loads entity according to its id, then apply given Consumer on it, and updates the result. Its a shortcut for who wants to apply some modifications on an entity without having to explicitly load it. Hence this can be seen as a helper for the Command Pattern where whole entity may not be given by the Command, but only properties that must be modified.
* `EntityPersister.updateById(id)` will update given entity with an update order on its primary key. As a difference with `update(entity)` it doesn't take optimistic lock into account
* `EntityPersister.deleteById(id)` will delete given entity with a delete order on its primary key. As a difference with `delete(entity)` it doesn't take optimistic lock into account

