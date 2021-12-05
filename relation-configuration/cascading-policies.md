# Cascading policies

Stalactite defines 4 types of cascade policy, they can all be applied to one-to-one or one-to-many relation. Please have a look at the table below to learn what each of them do:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Policy</th>
      <th style="text-align:left">On root insert</th>
      <th style="text-align:left">On root update</th>
      <th style="text-align:left">On root deletion</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">ALL</td>
      <td style="text-align:left">target is inserted if not already, or updated</td>
      <td style="text-align:left">target is updated, not deleted if deassociated</td>
      <td style="text-align:left">target is not deleted</td>
    </tr>
    <tr>
      <td style="text-align:left">ALL_ORPHAN_REMOVAL</td>
      <td style="text-align:left">target is inserted if not already or updated</td>
      <td style="text-align:left">target is updated, and deleted is deassociated</td>
      <td style="text-align:left">target is deleted</td>
    </tr>
    <tr>
      <td style="text-align:left">READ_ONLY</td>
      <td style="text-align:left">target is not inserted nor updated</td>
      <td style="text-align:left">target is not updated nor deleted</td>
      <td style="text-align:left">target is not deleted</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>ASSOCIATION_ONLY</p>
        <p></p>
      </td>
      <td style="text-align:left">targets and association table records are inserted</td>
      <td style="text-align:left">targets and association table records are updated (records may be deleted
        if target is deassociated)</td>
      <td style="text-align:left">targets are not deleted but association table record are deleted</td>
    </tr>
  </tbody>
</table>

{% hint style="info" %}
**ASSOCIATION\_ONLY particular case**

Please note that this policy is only relevent for one-to-many relation with association table.

This policy is intended to be used when targets shouldn't be inserted nor deleted but association must be maintained.
{% endhint %}

