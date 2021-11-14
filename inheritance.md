# Inheritance

Stalactite supports 2 types of inheritance : mapped super class and inheritance mapped with table relation.

### Mapped super class

It's usually a good pratice to mutualize persistence mapping of data. One can do it with a class containing shared state, which will be inherited by classes that don't need to define again this mapping. The inherinting classes will target their own table because it won't be dictated by super class. As an exemple, we can have a `Vehicle`class inherited by `Car`and `Truk`: `Car`s will be stored in the `Car`table with all their data, and `Truk`s will have their own data stored in the `Truk`table, independently.

{% hint style="info" %}
With mapped super class, only suclasses are allowed to define identifier policy.
{% endhint %}

Such a mapping can be defined as this :

```java
MappingEase.entityBuilder(Car.class, Long.class)
	// concrete class defines id
	.add(Car::getId).identifier(IdentifierPolicy.AFTER_INSERT)
	.add(Car::getModel)
	.mapSuperClass(MappingEase
			.embeddableBuilder(Vehicle.class)
			.add(Vehicle::getColor))
	.build(persistenceContext);
	
MappingEase.entityBuilder(Truk.class, Long.class)
	// concrete class defines id
	.add(Truk::getId).identifier(IdentifierPolicy.AFTER_INSERT)
	.add(Truk::getEngine)
	.mapSuperClass(MappingEase
			.embeddableBuilder(Vehicle.class)
			.add(Vehicle::getColor))
	.build(persistenceContext);
```

{% hint style="danger" %}
#### Mapped super class and before-insert identifier policy

With before-insert identifier policy user is left the choice to use any kind of sequence, and even reuse an already used sequence, hence creating a shared identifier pool for a mapped super class. This is technically allowed but is considered a \(very\) bad practice because it goes against the mapped super class principle which targets more technical usage than a real inheritance goal. For such sequence reuse, prefer real inheritance mapping.
{% endhint %}

### Inheritance with shared identifier

#### Table-per-class

By design Stalactite supports _table-per-class_ mapping because one can map all properties of a class through all its method references to a single table, but then you'll have to take care about identifier sequence reuse. For helping to such sharing one should use the `mapInheritance(..)` method :

```java
EntityMappingConfiguration<Vehicle, Identifier<Long>> inheritanceConfiguration = MappingEase
      .entityBuilder(Vehicle.class, Long.class)
      // mapped super class defines id
      .add(Vehicle::getId).identifier(IdentifierPolicy.ALREADY_ASSIGNED)
      .getConfiguration();

Persister<Car, Identifier<Long>, ?> carPersister = MappingEase.entityBuilder(Car.class, LONG_TYPE)
      .add(Car::getModel)
      .add(Car::getColor)
      .mapInheritance(inheritanceConfiguration)
      .build(persistenceContext);
```



But you may pay attention to identifier policy : for instance you can't obtain shared identifier if you choose after-insert policy \(a database-triggered value generator\) because identifier won't be shared accross other sibling entity classes since they'll target other table. As a consequence, **for a table-per-class inheritance policy one should use**

* already-assigned identifier
* or before-insert identifier

But in both cases, you must take care to apply a generator sharing values accross siblings and subtypes.

#### Single-table

Stalactite also supports _single-table_ mapping because, as it does for _table-per-class_, one can map all properties of a class through all their method references, but instead of using the `build(PersistenceContext)` method to obtain the final `Persister`, you're allowed to target a defined table thanks to the method `build(PersistenceContext, Table)`:

```java
MappingEase.entityBuilder(Car.class, Long.class)
	// concrete class defines id
	.add(Car::getId).identifier(IdentifierPolicy.AFTER_INSERT)
	.add(Car::getModel)
	.build(persistenceContext, new Table("Vehicle"));
	
MappingEase.entityBuilder(Truk.class, Long.class)
	// concrete class defines id
	.add(Truk::getId).identifier(IdentifierPolicy.AFTER_INSERT)
	.add(Truk::getEngine)
	.build(persistenceContext, new Table("Vehicle"));
```

As a difference with _table-per-class_, after-insert identifier policy is a fine choice because you're sure to share identifier sequence. For other policies, same warning applies as for _table-per-class_ : share your sequence.

{% hint style="danger" %}
Sharing mapping on a same table can leads to constraint integrity violation : if a column is mandatory for an entity, then it should be the same for an entity that shares the table, or constraint must be removed. Since Stalactite is "decentralized" \(mappings doesn't see each other\) as a difference with Hibernate \(or JPA\), it can't guaranty this mecanism \(removing constraint for instance\) and this choice is let to user \(who is encouraged to remove any mandatory column when using single-table mapping !\).
{% endhint %}

#### Joined-table

To create a _joined-table_ inheritance type, one must use the `withJoinedTable()` method after `mapInheritance(..)` :

```java
EntityMappingConfiguration<Vehicle, Identifier<Long>> inheritanceConfiguration = MappingEase
		.entityBuilder(Vehicle.class, Long.class)
		// mapped super class defines id
		.add(Vehicle::getId).identifier(IdentifierPolicy.ALREADY_ASSIGNED)
		.add(Vehicle::getColor)
		.getConfiguration();

Persister<Car, Identifier<Long>, ?> carPersister = MappingEase.entityBuilder(Car.class, LONG_TYPE)
		.add(Car::getModel)
		.mapInheritance(inheritanceConfiguration)
			.withJoinedTable()
		.build(persistenceContext, mappedSuperClassData.carTable);
```



