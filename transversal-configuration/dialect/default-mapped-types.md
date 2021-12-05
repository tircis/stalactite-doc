# Default mapped types

Dialects define the way simple types, meaning having only one value, are mapped onto a column. So, for it we needs 2 things :&#x20;

* a column SQL type, only required for database schema generation
* the way the column can be read and written, respectively from a ResultSet and a Statement



### Specifying SQL type

The former is accessible through `Dialect.getSqlTypeRegistry()` which gives you several put(..) methods either to define a SQL type for a Java class, or for a particular `Column`, in you need to override its SQL type. For example, supposing you need to store a JDK `AtomicInteger` (it has only one `int` internal value) :

```java
Dialect myDialect = new Dialect();
myDialect.getSqlTypeRegistry().put(AtomicInteger.class, "integer");
```

You can do the same but exceptionnaly for a Column :

```java
Dialect myDialect = new Dialect();
Table myTable = new Table("myTable");
Column<AtomicIntger> myAtomicColumn = myTable.addColumn("counter", AtomicInteger.class);
myDialect.getSqlTypeRegistry().put(myAtomicColumn, "integer");
```

{% hint style="success" %}
Stalactite declares some default SQL type in `Dialect` through a `DefaultTypeMapping`. It is expected to override this class to let one overwrite a SQL type mapping for a particular DB vendor and/or version, then it may be given to a dedicated Dialect class that should be [registered](registering-its-own-dialect.md).
{% endhint %}

{% hint style="info" %}
As a reminder, don't forget that defining SQL types is only necessary for schema generation. Hence it is not required if you don't use Stalactite for schema generation.
{% endhint %}

### Specifying how to read an write types

It can be interesting to be able to specify a particular way of reading and writing a simple Java type  if you need to adapt it (for compatibility with an RDBMS or business rule reason). To do it you must access the `ColumnBinderRegistry` by using `dialect.getColumnBinderRegistry()`, then you'll use `register(..)` methods  that either takes a `Class` or a `Column` as first argument (to define it for a particular `Column` requirement), and a `ParameterBinder` as second one. Here stays the definition since you'll need to define 2 methods :&#x20;

* `doGet(ResultSet resultSet, String columnName)`
* `set(PreparedStatement preparedStatement, int valueIndex, I value)`

Here is an example of all that process :

```java
ParameterBinder<User> userBinder = new ParameterBinder<User>() {
    @Override
    public User doGet(ResultSet resultSet, String columnName) throws SQLException {
        User user = new User();
        user.setName(resultSet.getString(columnName));
        return user;
    }
			
    @Override
    public void set(PreparedStatement statement, int valueIndex, User value) throws SQLException {
        statement.setString(valueIndex, value.getName());
    }
};
		
myDialect.getColumnBinderRegistry().register(User.class, userBinder);
// database SQL type should be adapted too (not direct purpose of this example)
myDialect.getSqlTypeRegistry().put(User.class, "varchar(255)");
```



{% hint style="success" %}
By default Stalactite declares some binders : the ones that are bounded to `ResultSet` getXXX(..) and `PreparedStatement` setXXX(...) methods, and some others like UUID, Path, File, etc. See `ParameterBinderRegistry` for full list.
{% endhint %}
