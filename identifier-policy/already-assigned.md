# Already assigned

As its name says, this policy means that entity id is already set, meaning by user code, and shouldn’t be managed by Stalactite. Meanwhile Stalactite needs to know entity persistent state in order to apply right SQL order (insert or update), so this policy requires to be given 2 methods:

* one that lets Stalactite knows if entity already exists in database
* one that lets Stalactite informs that entity has been inserted

An example of a simple (even basic) implementation of those methods would be a `boolean` stored in entity class, but one may also implement it through some cache.

Please note that Stalactite will also invoke the second method right after entity loading to take into account transient state management systems. Hence an implementation based on HTTP request life cycle is possible, as well as a transient `boolean` field in the entity (as mentioned above).

Here is an example with state stored on entity :

```java
MappingEase.entityBuilder(Car.class, long.class)
    .add(Car::getId).identifier(IdentifierPolicy.alreadyAssigned(Car::markAsPersisted, Car::isPersisted))
    .add(Car::getModel)
```

{% hint style="success" %}
This policy accepts any type of identifier. For instance UUID is a good candidate, but a more business one can also be used.
{% endhint %}
