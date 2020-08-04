# Don't create "a Persister to rule them all"

## Assessment

As your project grows, and features are added, developpers tend to add relations to the existing persistence context, as they would do by annotating classes of a classical JPA application.

In the same time, developpers also tend to fetch relations when it is necessary to their feature, but they hardly do it chirurgically : either by using EAGER annotation option, or fetching relation in a quite-centralized repository method.

This combo leads to an ever-growing memory footprit since relations are always loaded, even if their properties are not used in all situations.

## Any solution ?

### Bounded context

A better approach would require to create aggregates \(or bounded contexts\) dedicated to your "local" feature, as in Command and Query Separation \(CQS\) architecture, implying dedicated JPA persistence contexts for instance. Then those must be named and wisely referenced in code, for instance with the Spring @Qualifier annotation.

### JPA @NamedEntityGraph to the rescue ?

Another option would use the JPA `@NamedEntityGraph` annotation to configure some graph-loading contexts. But this as a drawback : you're still using a unique entity graph that is partially loaded, which doesn't help developper to know which relation is loaded or not.

{% hint style="success" %}
Stalactite promotes the aggregate approach because **you can visualize the graph complexity** when you write your mapping : hundreds lines of mapping would warn you about your graph inflation.

As a second advantage, by chaining your configuration writing, one may be less tempted to persist an intermediary entity and would **always access your graph by its root**, which also implies a no-surprise graph loading \(no additional surprising select\).
{% endhint %}



