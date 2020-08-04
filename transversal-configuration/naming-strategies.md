# Naming strategies

### Table naming strategy

One may require to give a different policy than the default one which takes mapped class name as table name. To do this you should use `tableNamingStrategy(..)` when building your mapping, by passing it an implementation of `TableNamingStrategy`.

{% hint style="info" %}
Default table naming strategy is TableNamingStrategy.DEFAULT
{% endhint %}

### Column naming strategy

The default column naming strategy is to use the mapped property name as the one for the column. To change it, one should use `columnNamingStrategy(..)` when building your mapping, by passing it an implementation of `ColumnNamingStrategy`.

{% hint style="info" %}
Default column naming strategy is ColumnNamingStrategy.DEFAULT
{% endhint %}

### Foreign key and join column naming strategy

Foreign keys and join columns can be named differently than the default strategy by applying some new `ForeignKeyNamingStrategy` and `ColumnNamingStrategy` during mapping with the respective `foreignKeyNamingStrategy(..)` and `joinColumnNamingStrategy(..)` methods.

{% hint style="info" %}
Default foreign key naming strategy is ForeignKeyNamingStrategy.DEFAULT

Default join column naming strategy is ColumnNamingStrategy.JOIN\_DEFAULT

ForeignKeyNamingStrategy.HASH and ForeignKeyNamingStrategy.HIBERNATE are also available
{% endhint %}

### Association table naming strategy

When mapping `List` association, you may need to change association table naming strategy, then use the method `associationTableNamingStrategy(..)` with an instance of `AssociationTableNamingStrategy` as argument.

{% hint style="info" %}
Default association table naming strategy is AssociationTableNamingStrategy.DEFAULT
{% endhint %}



