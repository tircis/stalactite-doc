# Multiple entities targeting same table

If you deal with some legacy schema or want to separate some domains of your application, it is perfeclty valid for Stalactite to target same table with different entities. Of course you'll have to manage separation of concern of your entities : don't map same column twice, nor identifier.

Hence if you have some huge table with many columns you may decide to split its naturally-mapped entity into several ones, and eventually map some has read-only for some domain context.

