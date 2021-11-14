# Persistence listeners

Stalactite provides a listener for each persistence action :&#x20;

* InsertListener
* UpdateListener
* SelectListener
* DeleteListener

They all can be given at a Persister through dedicated `addListener()` methods.

TODO: add code example, mention ListenerSupport classes

{% hint style="info" %}
Please note that while using ORM module, you’re only able to add listeners to root entity since it’s the only available persister, as a consequence you’ll only be notified with root entities as argument.
{% endhint %}

