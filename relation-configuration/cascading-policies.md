# Cascading policies

Stalactite defines 4 types of cascade policy, they can all be applied to one-to-one or one-to-many relation. Please have a look at the table below to learn what each of them do:

| Policy                         | On root insert                                     | On root update                                                                                       | On root deletion                                                 |
| ------------------------------ | -------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| ALL                            | target is inserted if not already, or updated      | target is updated, not deleted if deassociated                                                       | target is not deleted                                            |
| ALL\_ORPHAN\_REMOVAL           | target is inserted if not already, or updated      | target is updated, and deleted if deassociated                                                       | target is deleted                                                |
| READ\_ONLY                     | target is not inserted nor updated                 | target is not updated nor deleted                                                                    | target is not deleted                                            |
| <p>ASSOCIATION_ONLY</p><p></p> | targets and association table records are inserted | targets and association table records are updated (records may be deleted if target is deassociated) | targets are not deleted but association table record are deleted |

{% hint style="info" %}
**ASSOCIATION\_ONLY particular case**

Please note that this policy is only relevent for one-to-many relation with association table.

This policy is intended to be used when targets shouldn't be inserted nor deleted but association must be maintained.
{% endhint %}
