# On Insert Trigger for Default Date

We may not be too successful with setting a default sql value for a DateTime column upon its table's creation, so this is just a workaround. The SQL is:

```
CREATE TRIGGER `{schema}`.`{tableName}_BEFORE_INSERT` BEFORE INSERT ON `{tableName}` FOR EACH ROW
BEGIN
	SET NEW.CreatedDate = NOW();
END
```

Let's assume that `{tableName}` has a DateTime-valued column called `CreatedDate`. The column was created with nullable set to true so that we could insert
records to the table before creating this trigger. The line `SET NEW.CreatedDate = NOW();` is what will set the new record's `CreatedDate` value to the
current date and time, as returned by the built-infunction `NOW()`. The `NEW.` refers to the instance of the record about to be inserted to the table.