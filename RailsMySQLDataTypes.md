Auditing Rails MySQL DB schemas, the Rails schema file displays a particular field as `int, limit: 4` with the 4 set as default (not specified in the migration file). Looking at the production DB in a MySQL db console shows it as `Int(11)`. This field is actually provided by a MySQL view from a different DB, which shows it as `Int(4)`. What's going on and is this a problem?

```
create_table 'example' do |t|
  t.integer :int                 # int (4 bytes, max 2,147,483,647)
  t.integer :int1, :limit => 1   # tinyint (1 byte, -128 to 127)
  t.integer :int2, :limit => 2   # smallint (2 bytes, max 32,767)
  t.integer :int3, :limit => 3   # mediumint (3 bytes, max 8,388,607)
  t.integer :int4, :limit => 4   # int (4 bytes)
  t.integer :int5, :limit => 5   # bigint (8 bytes, max 9,223,372,036,854,775,807)
  t.integer :int8, :limit => 8   # bigint (8 bytes)
  t.integer :int11, :limit => 11 # int (4 bytes)
end
```

an `int` is always a size of 4 bytes, which enables a maximum number close to 2.15 billion (and not 9999!). Rails sets a `limit: 4` by default on an integer, and stores this as `Int(11)` - https://github.com/rails/rails/blob/ca5a35da373262e07862a75a670a7bf90b77e5c2/activerecord/lib/active_record/connection_adapters/abstract_mysql_adapter.rb#L651 for compatibility with MySQL.

Therefore, the number inside the brackets has nothing to do with limiting the size of the value, but simply the display 'width' for it; 

> MySQL supports an extension for optionally specifying the display width of integer data types in parentheses following the base keyword for the type. For example, INT(4) specifies an INT with a display width of four digits. This optional display width may be used by applications to display integer values having a width less than the width specified for the column by left-padding them with spaces. (That is, this width is present in the metadata returned with result sets. Whether it is used or not is up to the application.)

which doesn't really mean much unless you're using `ZEROFILL`, which will pad out numbers to fit the width specified. 
For example: If you're using ZEROFILL on a column that is set to INT(5) and the number 78 is inserted, MySQL will pad that value with zeros until the number satisfies the number in brackets. i.e. 78 will become 00078
