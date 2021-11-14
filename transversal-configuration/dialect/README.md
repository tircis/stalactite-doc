# Dialect

Dialects aim at adapting SQL to database vendor particularities, hence it is expected to have one Dialect per vendor and even per database version.

For example one can change generated DML (insert, update, delete and select orders) and overall DDL (table, colum, foreign key creation) by overriding method `newDDLGenerator()`.

