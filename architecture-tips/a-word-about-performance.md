# A word about performance

Stalactite uses JDBC batch for inserts, updates and deletes. But users have to keep in mind that inserts performance can be corrupted by identifier policy.

### Take caution to after-insert identifier policy and massive insert

If you use Stalactite to insert values into a single table then you don't have to worry so much about identifier policy. But if you're using Stalactite much more to help you persist an object graph and expect good performance then you may encouter some trouble if you used after-insert policy.

JDBC batch is used on PreparedStatement, which then expects to have all its values defined, so in case of  relational object, and in particular with one-to-one owned by source entity, it should know target ids before they're persisted to fill owning column, which is impossible with after-insert policy on target entities. As a consequence, in such case, Stalactite should persist target entities before root, and extends this mecanism to the whole graph. As a consequence, to benefit from JDBC batch, after-insert policy needs a different graph iteration algorithm than other policies. It may be possible, but not done yet ! 

