---
description: How to build beans from some columns of only one table
---

# Targeting one table

## Quickly building a bean from one Table

For simple case of building a bean from only one table, Stalactite provides the `PersistenceContext.select(..)` convenient method :

* the first argument is your bean factory (constructor or any factory method)
* other arguments are columns to be appended to the select clause (they are expected to be from same table, no join will be performed).

The factory (usually your bean constructor) will be given column values, hence its parameters must match `Column`s Java types. Here is a basic example :

```java
// Supposing a MyBean class with a one-arg constructor
Table myTable = new Table("MyTable");
Column<Table, Long> idColumn = myTable.addColumn("id", long.class);
List<MyBean> myBeans = persistenceContext.select(MyBean::new, idColumn);
```

{% hint style="info" %}
Note that one bean per row will be created : at this stage no bean graph will be built.
{% endhint %}

Of course, if your bean has a constructor with several arguments, some variations of the `select(..)` method exist with extra `Column` argument up to 3

```java
// Supposing a Person class with a 3-args constructor
Table personTable = new Table("Person");
Column<Table, Long> idColumn = personTable.addColumn("id", long.class);
Column<Table, String> nameColumn = personTable.addColumn("name", String.class);
Column<Table, Integer> ageColumn = personTable.addColumn("age", int.class);
List<Person> persons = persistenceContext.select(Person::new, idColumn, nameColumn, ageColumn);
```

## **Filling beans outside of constructor**

Some methods whose signatures are close to above ones are available with an extra parameter that allows bean properties fullfilment outside of constructor : an extra Consumer argument lets you add Columns and consume them. Here is an example :

```java
// Supposing a Person class with a 1-arg constructor and 2 properties 'name' and 'age' with their respective setter
Table personTable = new Table("Person");
Column<Table, Long> idColumn = personTable.addColumn("id", long.class);
Column<Table, String> nameColumn = personTable.addColumn("name", String.class);
Column<Table, Integer> ageColumn = personTable.addColumn("age", int.class);
List<Person> persons = persistenceContext.select(Person::new, idColumn, select -> select
                                                                      .add(nameColumn, Person::setName)
                                                                      .add(ageColumn, Person::setAge));
```

## **Filtering query**

Possibility to filter table is available as an extra `Consumer` of `CriteriaChain` that allows you to chain and / or criteria that will be appended to SQL where clause. Please note that columns should still be the one of same table and select clause (else it would generate an SQL targeting several tables without joining them), as `select(..)` method focuses on that use case.

```java
// Supposing a Person class with a 1-arg constructor and 2 properties 'name' and 'age' with their respective setter
Table personTable = new Table("Person");
Column<Table, Long> idColumn = personTable.addColumn("id", long.class);
Column<Table, String> nameColumn = personTable.addColumn("name", String.class);
Column<Table, Integer> ageColumn = personTable.addColumn("age", int.class);
List<Person> persons = persistenceContext.select(Person::new, idColumn,
    select -> select
    .add(nameColumn, Person::setName)
    .add(ageColumn, Person::setAge),
    where -> where
    .and(nameColumn, Operators.startsWith("Bob"))
    .and(ageColumn, Operators.gt(30)));
```

#### ****

