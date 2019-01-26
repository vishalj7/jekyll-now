---
layout: post
title: Introduction to Sqoop
tags: Hadoop Sqoop
---

### What is Sqoop?
Sqoop is an application that allows you to connect to a database like Oracle, MySQL to HDFS. It can import a single table, all tables or a partial table into HDFS. The data can be imported in a delimited text, SequenceFile or an Avro format. Sqoop can also export data from HDFS to a database.

Sqoop uses MapReduce to import and export data into and out of HDFS which provides parallel operation as well as fault tolerance. Sqoop will read the table row by row into HDFS and the output of this input process is a set of files containing the data that was imported. As Sqoop uses MapReduce to perform this operation the output will be multiple files.

When you execute a Sqoop import job a Java class is created which encapsulates one row of the imported table so this means a class will have all the columns within the table in this class kind of like having its schema. This class is used by Sqoop and the class can serialise and deserialise data from and to SequenceFile format, the class can also be used to parse delimited text format of a record. With these classes you can develop your MapReduce application which can then be execute on HDFS. You can use any other tool to parse the record yourself.

Sqoop can also export data from HDFS to a database and the way it does this is by reading a set of delimited text files and parsing each file into records and then inserts the records as new rows in the target database and table. 

With Sqoop you can control the row ranges and/or control the columns that you would like to import/export. There are other cutomisable options which you can choose for example, the delimiter, any escape character.

#### Note that the default file output format for Sqoop is a comma delimited text file. 

### Some database inspect commands

Sqoop has some tools which allow you to 'see' the schema of the database and even the schema for the tables. Later on we will see examples of the commands and its output.

```
sqoop list-databases --connect jdbc:mysql://quickstart.cloudera:3306 --username root --password cloudera

Output 

17/04/21 11:47:47 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6-cdh5.8.0
17/04/21 11:47:47 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
17/04/21 11:47:47 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
information_schema
cm
firehose
hue
metastore
mysql
nav
navms
oozie
retail_db
rman
sentry
```

```
sqoop list-tables --connect jdbc:mysql://quickstart.cloudera:3306/retail_db --username root --password cloudera

Output

17/04/21 11:49:58 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6-cdh5.8.0
17/04/21 11:49:58 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
17/04/21 11:49:58 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
categories
customers
departments
order_items
orders
products
```


### Using Generic and Specific Arguments

Sqoop has a number of tools which can be used to import, export, list tables etc and each of these tools have their own arguments that can be specified in the Sqoop command. Along with specific arguments Sqoop has generic arguments that are required in the command and these arguments (if you need them) need to be after the tool but before the tool's arguments. Some of the generic arguments are -conf, -D, -fs etc.

One indicator between the two argument types is that generic arguments have a single '-' where specific arguments has two and the only exception is when a specific argument is a single character. 


### Sqoop Import

We will now look at the the Sqoop import tool which is used to import a table from different types databases into HDFS. The import is performed by Map-only jobs and each row from the table will be represented as a record in HDFS and depending on the file type argument you pass it will be in text, Avro or SequenceFile. 

To import a table you need to specify the connection details by using the argument - -connect. The argument value needs to describe the server and database, you may also want to specify the port number. As with most databases you would need to give the username and password, use - -username and - -password with the values for the database. There are a number of differentways to supply the password both secure and non-secure, which we will go through now.

```
sqoop import --connect jdbc:mysql://quickstart.cloudera:3306/retail_db --username root --password cloudera --table orders --warehouse-dir table_warehouse/
```

You can also save the password in a file which has limited permissions, and then using the - -password-file argument to provide the locagtion of the file.

Sqoop supports a number of databases and for the ones that have jdbc:mysql:// are handled automatically but for some database types the relevant .jar is not installed and the .jar needs to be installed. You would need to download the relevant JDBC driver that you require for your database and the place the .jar file in the ${Sqoop_HOME}/lib. Each driver .jar has a specified driver class and it is this class that needs to be defined when stating the type of driver that needs to be used to connect to your requested database.

```
sqoop import --driver com.mysql.jdbc.Driver --connect jdbc:mysql://.....
``` 

There are lot of options which can be specified when running the Sqoop command and I have listed some below:

| Argument | Description|
|---|:---:|
|- -append |This will append data to existing table in HDFS
|- -as-avrodatafile | The data will be stored as an Avro file in HDFS 
|- -as-textfile | The data will be stored as a text file default file format (default delimiter = comma separated)                    
|- -target-dir <dir_name> |Here you can specify the destination folder where you want Sqoop to store the data
|- -warehouse-dir <dir_name>|Here you specify the parent folder where Sqoop will stored the data
|- -table |Specify which table to import
|- -hive-import |Import the table into Hive

Where you see <...>, is where you need to provide the actual value.


##### Note - you can't use target-dir and warehouse-dir in the same command as both are similar. The difference is that with target-dir it is a full directory path and the files will be placed under the specified folder where as warehouse-dir is a parent directory where the files will be placed in a folder named after the table and this table folder will be under the directory you specify as the warehouse-dir (the parent directory).


#### Import Only Selected data

With Sqoop you can restrict what data is actually imported into HDFS such as selecting only a table, or certain columns from a table and you can even limit number of rows and only select rows that meet a certain condition. Below are some options that can be used to restrict the data that is imported from a table:

| Argument | Description|
|---|:---:|
|- -columns '<...,...>'| Here you specify which columns to select when importing from a table (this needs to be in a comma separated list)|
|- -where '<...>'| Using the where clause just like in SQL by stating a condition that needs to be met|
|- -query '<...>'| Here you can use a select query to limit the data that is imported|

#### Note - When using the - -query argument, you need to specify the - -target-dir and the '$CONDITIONS'.

```
sqoop import --connect jdbc:mysql://quickstart.cloudera:3306/retail_db --username root --password cloudera --query 'SELECT * FROM products WHERE product_category_id=7 AND $CONDITIONS' --target-dir id7/test --m 1
```
Using '- -m 1' specifies the number of mappers (1) and since we are not split the processing we do not need to use '- -split-by'

If you want to import the data using a query and want to paralellise the mapper tasks so that each mapper tasks executes part of a query then you need to add a column to split by. Sqoop will use the ranges from the split-by column to split the data evenly and replace the $CONDITIONS with the range values. For example a column that you specify is called 'Age' with the minimum value of 0 and the maximum value of 100 and you select 4 map tasks then task 1 would receive data where 'Age' is between 0 and 24 and the next task would get data where 'Age' is between 25 and 49 and so on.

If you want your Sqoop to run parallel and you need to specify the number of mapper tasks to uses to perform the import. The default number of mapper tasks is 4 and often more mapper tasks perform better than the default 4 but sometimes less performs better and this depends on the data that you are importing. When Sqoop performs paralell imports, Sqoop needs something to split the data by so that it can try and give even as possible split of the data to each map task. 

Sqoop by default splits by the primary key of a table if it is present but if your data has a small range of values for the primary key, then it will cause the mapper tasks to have an uneven tasks as some mapper tasks may have a lot less than other mapper tasks. So to avoid this you would need chose and specify a column which can be used to split the data, the way to do this is to use the - -split-by option and then specify the column that you want Sqoop to use to split the data evenly for the mapper tasks. 

```
sqoop import --connect jdbc:mysql://quickstart.cloudera:3306/retail_db --username root --password cloudera --query 'SELECT * FROM products WHERE product_category_id=7 AND $CONDITIONS' --target-dir id7/test --split-by 'product_id'
```

#### Note that if the table that is being imported has no primary key or the split-by column has not been speficied then the import will fail unless the number of mappers is specified as 1 using the option '- -num-mappers 1' or '- -m 1'. 

### Hive Import 

Sqoop can import the data from a database into HDFS as files but if you have a Hive metastore you can specify that Sqoop imports the data into a Hive table using '- -hive-import', Sqoop will run an 'CREATE TABLE' statement based on the data you are importing. In case the table already exists you can specify '- -hive-overwrite' so if the table already exists then Sqoop will overwrite the table. 

First Sqoop will import the data files into HDFS and then Sqoop will generate and run a 'CREATE TABLE' statement. After that Sqoop will run a 'LOAD DATA INPATH' statement which will move the data files into the Hive warehouse directory. 

#### Note that you will not be able to use '- -as-avrodatafile' or '- -as-sequencefile' with '- -hive-import'

Sqoop will import NULL values as the string 'null' by default but Hive uses '\N' to specify Null values so commands like 'IS NULL' in Hive when querying will not work as expected as the NULL value is in the form of '\N'. So you would use the option '- -null-string' to specify what the null value should be stored as for string columns and '- -null-non-string' to specify what the null value should be stored as for non string columns. 

The original table name will be used as the table in Hive but if you want to use a different name you can use the option '- -hive-table' to specify an new name.

Also you can partition the data as it is being imported by using 'hive-partition-key' to specify a column to partition on and '- -hive-partition-value' to specify what value to partition on.

```
sqoop import --connect jdbc:mysql://quickstart.cloudera:3306/retail_db --username root --password cloudera --table orders --hive-import --hive-table renamed_to_hive_orders 

beeline -u jdbc:hive2://quickstart.cloudera:10000/default -n cloudera -p cloudera

select count(*) from default.renamed_to_hive_orders;

+--------+--+
|  _c0   |
+--------+--+
| 68883  |
+--------+--+
1 row selected (19.984 seconds)
```

##Sqoop Import All Tables

### Import All Tables

Sqoop has a tool that can import all the tables from a particular database intp HDFS. The tool is called 'import-all-tables' and the data is stored in a its relevant directory in HDFS where the directory name is based of the table name. 

For 'import-all-tables' to be most effective the below must be met:

+ You can not use the '- -where' clause to limit the data to want to import
+ You must import all of the columns for every table (but you don't have to import every table)
+ Every table must have a single primary key column other wise '- -autoreset-to-one-mapper' must be used so that only one mapper task is used if there is no single primary column

Most of the options that can be used for the 'import' tool is available for 'import-all-tables' and can be used in the same except for a few options which can not be used.
Some of them are listed below:

+ '- -table'
+ '- -columns'
+ '- -split-by'
+ '- -where'

One option that is available for 'import-all-tables' but is not available for 'import' is '- -exclude-tables' where you provide a comma seperated list of tables that you don't want to export.

```
sqoop import-all-tables --connect jdbc:mysql://quickstart.cloudera:3306/retail_db --username root --password cloudera --warehouse-dir all-tables/
```


### Sqoop Export 

The Sqoop export tool allows you to export the data files from HDFS to a RDBMS but the target table must exist in the target RDBMS as the files from HDFS are read and parsed into a set of data records and delimited by the user's specified value.

Sqoop will create 'INSERT' statements which are ran in the target database but it will create 'UPDATE' statements when you have specified '- -update-mode'. 

Sqoop export has very similar options to the import tool but you need to specifiy the '- -export-dir' which represents the folder where the data files exist that you want to export. The other option you need to specify is either '- -table' to specify the target table or '- -call' to specify the stored procedure you want to call.

Just like Sqoop import you can specify the columns that you want to export and you can change the order by using '- -columns' and providing a comma seperated list of columns names. If you don't include a column in the export, you need to make sure that excluded column in the target database has a default value or allows 'NULL' values. Otherwise the database will reject the imported data.

Below are some of the options from the Import tool that work for the Export tool:

+ Control the number of mappers using '-m'
+ Using the direct mode using '- -direct'
+ Specify the null values you want using '- -import-null-string' and '- -import-null-non-string'

If '- -import-null-string' is not specified then the string 'null' will be treated as 'NULL' and if '- -import-null-non-string' is not specified then string 'null' and empty values in non-string columns will be treated as 'NULL'.

Sqoop breaks down the export process into number of transactions and it is possible that one or more of these transactions can fail and when restarting the Sqoop job it may fail as there is already data there or you will get duplicate data into the target table. So what you can do is use the option '- -staging-table' which acts as a temp table in the target database to allow all of the data records to be put in this table before being exported to your database. 

This '- -staging-table' option should be created prior to running the export and it must be an identical copy of the target table. It also must be empty before running the export job or you can specify the option '- -clear-staging-table' which will delete any data before running the export job. 

Sqoop will create code to parse and interpretate records from the data that is to be exported but if the data files don't use the default delimiter (comma separated) or the newline character (\n) then you must specify these values in order for Sqoop to parse the data correctly. if you don't Sqoop won't be able to parse the data and the export will fail. 

```
CREATE TABLE demo_target (
id int,
date varchar(255),
customer_id int,
status varchar(255)
);

sqoop export --connect jdbc:mysql://quickstart.cloudera:3306/retail_db --username root --password cloudera --export-dir /user/hive/warehouse/renamed_to_hive_orders --table demo_target --input-fields-terminated-by '\001' --input-lines-terminated-by '\n'

select count(*) from retail_db.demo_target;
+----------+
| count(*) |
+----------+
|    68883 |
+----------+
```

### Insert or Update

If you want to add new rows to an existing table then by default Sqoop will run an 'INSERT' statement for each row so you do not have to specify any options for the insert. If the table contains a primary key column and it tries to insert a row that where the primary key value already exists then the 'INSERT' statement will fail causing the job to fail. You can update rows of data in the target table and you need to use the  '- -update-key' option and provide a column/s to use to match against existing records which those that match will be updated. Sqoop will create a 'UPDATE' statement for each record in the data and it will using the provided '- -update-key' as the where clause to use to match records in the target table and update those that match. 

#### Note - If no records are updated or multiple records are update with a single update statement then no errors will be returned. 

