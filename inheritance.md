# Inheritance

Stalactite supports 2 types of inheritance :&#x20;

* mapped super class, that only centralizes mapped properties, not id policy
* inheritance aimed at defining id policy

{% hint style="info" %}
In this chapter you'll get a persister per child class, hence you can't load all children entities of a mapped super class at once : for that see polymorphism.
{% endhint %}

### Mapped super class

To avoid describing several times same persistence mapping of a common parent class, you can describe it through an "embeddable" mapping and reference it in children classes mapping thanks to the `mapSuperClass(..)` method. Since this way of doing is just for skipping repeation of common mapping, it is not possible to define target table of super class as well as its persistence policy : both are only done by children classes.

As an example, we can have a `Vehicle`class inherited by `Car`and `Truk`: `Car`s will be stored in the `Car`table with all its data, and `Truk`s will have its own stored in the `Truk`table, independently. Such a mapping can be defined as this :

```java
FluentEmbeddableMappingBuilder<Vehicle> vehiclePersistenceConfigurer =
    MappingEase.embeddableBuilder(Vehicle.class)
	.add(Vehicle::getColor)


MappingEase.entityBuilder(Car.class, Long.class)
	// concrete class defines id
	.add(Car::getId).identifier(IdentifierPolicy.AFTER_INSERT)
	.add(Car::getModel)
	.mapSuperClass(vehiclePersistenceConfigurer
	.build(persistenceContext);
	
MappingEase.entityBuilder(Truk.class, Long.class)
	// concrete class defines id
	.add(Truk::getId).identifier(IdentifierPolicy.AFTER_INSERT)
	.add(Truk::getEngine)
	.mapSuperClass(vehiclePersistenceConfigurer)
	.build(persistenceContext);
```



### Inheritance to define identifier policy

As a difference with mapped super class, hereafter the shared parent mapping defines id policy. This inheritance is declared by the `mapInheritance(..)` method.

Here is a simple exemple where a configuration defines mapped properties of `Vehicle` class, as well as its identifier policy, then the `Car` class can benefit of it.

```java
EntityMappingConfiguration<Vehicle, Identifier<Long>> inheritanceConfiguration = MappingEase
      .entityBuilder(Vehicle.class, Long.class)
      // mapped super class defines id
      .add(Vehicle::getId).identifier(IdentifierPolicy.ALREADY_ASSIGNED)
      .add(Vehicle::getColor)
      .getConfiguration();

Persister<Car, Identifier<Long>, ?> carPersister = MappingEase
      .entityBuilder(Car.class, Long.class)
      .add(Car::getModel)
      .mapInheritance(inheritanceConfiguration)
      .build(persistenceContext);
```



By default mapped properties in parent configuration are stored in child class table, but you may want to store them in different tables through a join on primary key. For such a case one can simply ask for `withJoinedTables(..)` right after calling `mapInheritance(..)`. The method has 2 signatures, one without argument meaning that parent table will be named according to configured table naming strategy, and one of them taking the table on which parent class persistence must be done. Here is same example as previous one with this option activated :

```java
EntityMappingConfiguration<Vehicle, Identifier<Long>> inheritanceConfiguration = MappingEase
      .entityBuilder(Vehicle.class, Long.class)
      // mapped super class defines id
      .add(Vehicle::getId).identifier(IdentifierPolicy.ALREADY_ASSIGNED)
      .add(Vehicle::getColor)
      .getConfiguration();

Persister<Car, Identifier<Long>, ?> carPersister = MappingEase
      .entityBuilder(Car.class, Long.class)
      .add(Car::getModel)
      .mapInheritance(inheritanceConfiguration)
            .withJoinedTable(new Table("Vehicle"))
      .build(persistenceContext);
```
