---
layout: post
title: Avro Files
tags: 
- Avro
---
### What is Avro?
Avro is an efficient data serialisation framework and a RPC (Remote Procedural Calls) library. You can read and write data in Java, C, C++, C#, Python, PHP and other languages and then the data is serialised it uses a highly-optimised binary encoding. Avro specifies rules for schema evolution over time.

An Avro Container File is made up of the data and the schema for the data making it self-describing so this means it is easy for programs to dynamically understand the information stored in the Avro file. This makes the Avro file very portable as any program just needs to read and understand the schema to access/read the data. 

### What is data serialisation?

Data serialisation is a way of translating data into binary form so that it can be sent across the network or to store the translated data to disk. Where deserialisation is where data in binary format is translated into actual data rather than a binary format.

Avro utilises a compact binary data format which can be compressed and this results in a fast serialisation times. 

Note – that serialising the number ‘108125150’ as an int type takes up 4 bytes where storing it as a string will take up 9 bytes.

### Avro schema

Avro schemas are represented in JSON format and there are two Avro data types; Simple and Complex.

#### Simple – contains only one value


| Avro  |  Java Equivalent |  
|---|---|
| null  |  null |  
| boolean  | boolean  |  
| int  |  int |  
| long | long |
| float | float |
| double | double |
| bytes | java.nio.ByteBuffer |
| string | 	java.lang.CharSequence |



#### Complex

|Name	|Description|
|---|---|
|record|A user-defined type composed of one or more named fields|
|enum	| A specific set of values |
|array | Zero or more values of the same data type |
|map	| Set of key-value pairs; key is of string data type and value is specified data type |
|union	|Exactly one value matching a specified set of types |
|fixed |	A fixed number of 8-bit unsigned bytes |



The Avro schema is just a way to define the Avro data and it has its own standards of defining a schema which are stated below.

+ Namespace – name of the namespace in which the object resides
+ Type – the document type (default is record)
+ Name – the schema name
+ Fields – this an attribute array where the “column names” and its data types are specified


Below is an example of an Avro schema.

```json
{
    "type":"record",
    "name":"avro_schema",
    "namespace":"avro_schema",
    "fields":[
            {"name": "Name", "type":"string"},
            {"name": "Age", "type":"int"},
            {"name": "col_3", "type":"double"},
            {"name": "col_4", "type":"float"},
            {"name": "col_5", "type":"boolean"}
    ]
}
```

Avro supports setting default values in the schema.


Default example

```json
{
    "type":"record",
    "name":"default_example",
    "namespace":"default_example",
    "fields":[
            {"name":"First_Name", "type":"string"},
            {"name":"Last_Name", "type":"string"},
            {"name":"Language", "type":"string", "default":"English"}
    ]
}
```

Avro checks for null values when serialising the data and null values are only allowed when they have been specified in the schema. If you are using a default, the type must be listed first in the union. 

Null example

```json
{
    "type":"record",
    "name":"null_schema ",
    "namespace":"avro_schema",
    "fields":[
            {"name": "Name", "type":"string"},
            {"name": "Age", "type":"int"},
            {"name": "col_3", "type":["null", "double"]},
            {"name": "col_4", "type":["null", "float"]},
            {"name": "col_5", "type":"boolean"}
    ]
}
```

You can also use the ‘doc’ to give a description about that type and this works for every types.

```json
{
…
"fields":[
            {"name": "Name", "type":"string", 
            "doc":"Name of the user"},
…
}
```

### Avro supports compression 



### Avro Schema Evolution

Schemas of data will of course change and Avro supports these changes. The schema used to write the data is contained within the file and therefore when reading the data the schema is used to define the structure of the data.

If there is a need for the schema to change we can write the new data using a new schema but the problem with that is applications would not be able to read the old data. 

With changes to column names you can need to use “aliases” and reference the old column name. So that the old and new data can be read using this one schema. 

```json
{
…
"fields":[
            {"name": "User_Name", "type":"string", 
            "doc":"Name of the user", "aliases":["Name"] },
…
}
```

Where new fields are added, there is no previous data for the column so we need to state the default value where one doesn’t currently exist. 

```json
{
…
"fields":[
            {"name": "User_Name", "type":"string", 
            "doc":"Name of the user"},
            {"name":"Email", "type": ["null", "string"], 
            "default":"null"},
…
}
```

If you change an Avro file’s schema the changes below will not affect any application reading the file using the old Avro schema:

+ Adding, changing or removing the "doc" attribute.
+ Changing the field’s default value 
+ Adding a new field with a default value
+ Adding a aliases for a field
+ Promoting a field to a wider data type (eg, int to long)
+ Removing a field with a default value


The changes below would affect the application ability to use the old schema to read the old data if changes are made to the schema:

+ Creating a new field without a default value
+ Changing a field’s name with no aliases
+ Changing the record’s name or namespace
+ Removing a symbol from an enum
+ Removing a type from a union


### Avro Tool 


The Avro Tool allows you to examine an Avro files as it allows you to read the schema or the data and each Avro release contains an Avro Tools JAR file. If you want to read the data, you need to use ‘tojson’ and use ‘getschema’ to read the schema.
When using Avro “specific” records, you need first generate the Java code and one way to do this is to use the Avro Tool JAR file. You need to make sure that the output directory must already exist and that you generate the code to its own directory. 

