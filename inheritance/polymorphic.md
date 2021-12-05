# Polymorphic

In this chapter you'll learn how to define a persister that allows to persist subtypes of one parent class, and, at load time, creates instances whose type is deduced from data.



The entry point for it is the `mapPolymorphism(PolymorphismPolicy)` method : depending on the way you want to spread your data uppon tables, you'll have to give the right polymorphism policy which is it self built from one of the methods available on the `PolymorphismPolicy`type.

### **Persisting all siblings in same table : single-table**

Single-table policy is available from `PolymorphismPolicy.singleTable()` method, on which one must chain `addSubClass(..)` for each subtype to be managed by persister. This method takes a subentity configuration as argument, which is available through `MappingEase.subentityBuilder(..)`, as well as a discriminator value that allows to distinguish instance type from its siblings. Hereafter is an exemple :&#x20;

```java
EntityPersister<AbstractVehicle, Long> abstractVehiclePersister = MappingEase.entityBuilder(AbstractVehicle.class, Long.class)
    .add(AbstractVehicle::getId).identifier(ALREADY_ASSIGNED)
    .add(AbstravtVehicle::getColor)
    .mapPolymorphism(PolymorphismPolicy.<AbstractVehicle>singleTable()
        .addSubClass(MappingEase.subentityBuilder(Car.class)
            .add(Car::getModel), "CAR")
        .addSubClass(MappingEase.subentityBuilder(Bus.class)
            .add(Bus::getCapacity), "BUS")
    .build(persistenceContext);
```

The discriminator column name can be configured as a parameter of `singleTable()`, by default it is "DTYPE".

### **Persisting each sibling in its dedicated table : table-per-class**

Table-per-class policy is available from `PolymorphismPolicy.tablePerClass()` method, on which one must chain `addSubClass(..)` for each subtype to be managed by persister. This method takes subentity configuration as argument, which is available through `MappingEase.subentityBuilder(..)`. Hereafter is an exemple :&#x20;

```java
EntityPersister<AbstractVehicle, Long> abstractVehiclePersister = MappingEase.entityBuilder(AbstractVehicle.class, Long.class)
    .add(AbstractVehicle::getId).identifier(ALREADY_ASSIGNED)
    .add(AbstravtVehicle::getColor)
    .mapPolymorphism(PolymorphismPolicy.<AbstractVehicle>tablePerClass()
        .addSubClass(MappingEase.subentityBuilder(Car.class)
            .add(Car::getModel))
        .addSubClass(MappingEase.subentityBuilder(Bus.class)
            .add(Bus::getCapacity))
    .build(persistenceContext);
```

Child tables can be explicitly targeted with a `Table` extra parameter of the `addSubClass(..)` method.

### **Persisting parent properties in one table and siblings ones in dedicated ones : join-table**

Join-table policy is available from `PolymorphismPolicy.joinTable()` method, on which one must chain `addSubClass(..)` for each subtype to be managed by persister. This method takes subentity configuration as argument, which is available through `MappingEase.subentityBuilder(..)`. Hereafter is an exemple :&#x20;

```javascript
EntityPersister<AbstractVehicle, Long> abstractVehiclePersister = MappingEase.entityBuilder(AbstractVehicle.class, Long.class)
    .add(AbstractVehicle::getId).identifier(ALREADY_ASSIGNED)
    .add(AbstravtVehicle::getColor)
    .mapPolymorphism(PolymorphismPolicy.<AbstractVehicle>joinTable()
        .addSubClass(MappingEase.subentityBuilder(Car.class)
            .add(Car::getModel))
        .addSubClass(MappingEase.subentityBuilder(Bus.class)
            .add(Bus::getCapacity))
    .build(persistenceContext);
```

Joined tables can be explicitly targeted with a `Table` extra parameter of the `addSubClass(..)` method.
