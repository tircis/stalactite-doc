# Persistence listeners

Stalactite provides a listener for each persistence action :&#x20;

* InsertListener
* UpdateListener
* SelectListener
* DeleteListener

Each on these classes contains methods to be overriden for handling moments of their dedicated trigger :&#x20;

* before
* after
* error

They all can be given to an `EntityPersister`, which is the result of the `entityBuilder(..)...build(..)` method, through dedicated `addListener(..)` methods. Here is an example that manages timestamp of a Car :&#x20;



```java
EntityPersister<Car, Long> carPersister = entityBuilder(Car.class, Long.class)
    .add(Car::getId).identifier(afterInsert())
    .add(Car::getModel)
    .add(Car::getColor)
    .embed(Car::getTimestamp, embeddableBuilder(Timestamp.class)
        .add(Timestamp::getCreationDate)
        .add(Timestamp::getModificationDate))
    .build(persistenceContext);

carPersister.addInsertListener(new InsertListener<Car>() {
    @Override
    public void beforeInsert(Iterable<? extends Car> entities) {
        entities.forEach(e -> e.setTimestamp(new Timestamp()));
    }});
carPersister.addUpdateListener(new UpdateListener<Car>() {
    @Override
    public void beforeUpdate(Iterable<? extends Duo<? extends Car, ? extends Car>> payloads, boolean allColumnsStatement) {
          payloads.forEach(p -> p.getLeft().getTimestamp().setModificationDate(now()));
    }});
```





{% hint style="info" %}
Please note that while using ORM module, you’re only able to add listeners to root entity since it’s the only available persister, as a consequence you’ll only be notified with root entities as argument.
{% endhint %}

