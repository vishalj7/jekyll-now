---
layout: post
title: Sqoop In More Detail
---

### Sqoop In More Detail


### Options File 

If you use a Sqoop command over and over again you can write the arguments in a options file which can then be passed to the Sqoop command. In this file, each line would state the argument and now the next line state its value. You can have comments and blank line within this file but Sqoop will ignore these when running the command. Comments would require a '#' at the start of the line. You would use the argument - -options-file to specify that you are using the option file and then you would pass the location of the file.

Example file:
```
import
--connect
jdbc:mysql://localhost/db
--username
vishal
```

Where the command would look like this:
```
sqoop --options-file /usr/home/work/import.txt --table TEST
```

### Direct Import

Sqoop by default will use JDBC to import the data but for some databases there are other movement tools that allows the import to be in a high-performance fashion. For example MySQL has a tool called 'mysqldump' which can export data from MySQL to other systems very quickly. You would add the argument - -direct in your Sqoop command so that Sqoop will attempt to use the direct import channel. If the argument '- -' is given then any arguments after this are provided then these arguments are sent directly to the underlaying tool so in the case of MySQL the arguments will be passed to the 'mysqldump' tool.

```
example
```

### Incremental Loads

Sqoop allows you to use an incremental import mode which will only get the rows that are new compared to the previously imported rows. Using the argument '- -incremental' you can specify the type of incremental import to run, there are two types 'append' and 'lastmodified'. 

You use 'append' when the table you are importing contains new rows that are constantly being added and has an increasing id value. You would use '- -check-column' to specify which column is the row's id column. Sqoop will import rows where the value for a column (specified by using '- -check-column') is greater than the value specified by '- -last-value'.

```
sqoop import -table 
```

The other type is when updates happens to rows in the table that you want to import and a column containing a timestamp is updated to the time of the update. Sqoop will import rows where the timestamp value is for a column specified by using '- -check-column' is greater than the value specified by '- -last-value'.
