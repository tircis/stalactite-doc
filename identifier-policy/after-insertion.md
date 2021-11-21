# After insertion

This identifier policy uses database-generated id through auto-increment column. This policy only works with integer numbers such as `Long` or `Integer`.

```java
MappingEase.entityBuilder(Car.class, long.class)
    .add(Car::getId).identifier(IdentifierPolicy.AFTER_INSERT)
    .add(Car::getModel)
```
