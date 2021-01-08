---
title: File Formats
description: Hadoop file formats.
sequence: 03
---

{% include toc.html %}

## Key Concepts
### Overview
There's a number of things to consider when choosing which file format to use on a big data cluster.

### Compression
Compression is good because it reduces the amount of data that needs to be stored and transferred. However, files need to be splittable so they can be processed in parallel - you need to be able to process one partition of data independently from the others. Some compression algorithms will cause files to not be splittable.

Note if using a file format such as Avro or sequence files, the file will be splittable no matter what the compression codec (codec = compressor and decompressor). This is because these formats apply compression to file blocks rather than simply compressing the entire file (as is the case with text files). See [this post](https://stackoverflow.com/a/32430386/11550924) for more detail.

Whether the compressed file is splittable is one thing to keep in mind when choosing a compression codec . Other things include how compressed the file is (the degree of compression)  and the speed at which it is compressed.

Common compression codecs include gzip, bzip2, snappy and LZO.

### Schema Evolution
When using a schema it's important to keep in mind how easy it is to evolve that schema over time.

### Encoding
Keep in mind the differences between a binary and a text (human readable) encoding when looking at different file formats. Binary encodings are often used as they are more compact.

If you've never thought about this, don't be confused. Text (e.g. ASCII) is of course stored as binary. But say you want to store the number ```100 000 000```. The text encoding of this number will require 9 characters (9 bytes), but it could easily fit inside a 4 byte integer. 

## Popular Formats
### Text
Plain text files such as csv are portable and widely used. If using compression, a splittable compression algorthm (e.g. bzip2) should be used.

### Sequence
Sequence files are stored in binary format and each record is a key value pair. This approach makes them a suitable solution for the small file problem in Hadoop. The small file problem refers to the issue of too much NameNode memory being used when storing a lot of small files in HDFS. Using sequence files these small files can be combined into one large file and uniquely identified using the record key (i.e. key is filename, value is file contents). See [this post](https://blog.cloudera.com/the-small-files-problem/) for more details.

In addition sequence files support block level compression, which means that they are splittable even when using a non splittable codec.

### Avro
Avro also supports block level compression. It's not so much a format as a tool for serialization and data exchange.

To encode a record using Avro, a schema must be defined. For example:
```
{
    "type": "record",
    "name": "employee",
    "fields": [
        {"name": "firstName", "type": "string"},
        {"name": "lastName", "type": "string"},
        {"name": "nickName", "type": ["null", "string"], "default": null}
    ]
}
```

This defines an ```employee``` record with three fields - ```firstName```, ```lastName``` and ```nickName```. Most of it is self explanatory, but notice the type of ```nickName``` includes ```null```. This makes the field optional.

Using this schema, the record
```
{
    firstName: "Billy",
    lastName: "Smith",
    nickName: "Big b"
}
```

can be encoded into a relatively small number of bytes. This is because no field names need to be stored - only the values.

Later on the message can be read back with the same or a different schema. The advantage of using a tool like Avro is that schema evolution is easy. The basic idea is when the reader wants to decode a message, it compares its schema with the writer's. A few cases can arise:
- They're the same. Easy!
- There are fields in the reader's schema that aren't in the writer's. This is an issue of backwards compatibility. This is handled by putting a default value in newly added fields.
- There are fields in the writer's schema that aren't in the reader's. This is an issue of forwards compatibility. These fields can be ignored.

Writing Avro data using Spark is easy:
```
val employees = Seq(("Billy", "Smith", "Big b"))
val employeeCols = Seq("firstName", "middleName", "lastName")
val employeeDf = employees.toDF(employeeCols:_*)
employeeDf.write.format("avro").save("employees.avro")
```

Make sure spark-shell is started with the Avro dependency:
```
spark-shell --master yarn --deploy-mode client --packages org.apache.spark:spark-avro_2.11:2.4.6
```

Note the above command is for for Spark version 2.4.6 with the YARN cluster manager.

The data can be read back easily as well:
```
val employeeDf2 = spark.read.format("avro").load("employees.avro")
employeeDf2.show
```

You could also specify the reader's schema. Save the schema [above](#avro) as ```employee.avsc``` and then execute:
```
val employeeSchema = new Schema.Parser().parse(new File("employee.avsc"))
val employeeDf = spark.read.format("avro").option("avroSchema", schema.toString).load("employees.avro")
```

When I did this, I got ```error: not found: type File```. To get around that execute ```import java.io.File```.

### Parquet
Parquet is a columnar binary format. This means it stores records next to each other that are in the same column, not the same row. This is particularly useful when a small number of columns of a large table are being queried. In addition columnar formats can be compressed to a high degree given the similarity of data in the same column.

A similar example as in the [Avro](#avro) section is shown below for Parquet:
```
val employees = Seq(("Billy", "Smith", "Big b"))
val employeeCols = Seq("firstname", "middlename", "lastname")
val employeeDf = employees.toDF(employeeCols:_*)
employeeDf.write.parquet("employees.parquet")
val empoyeeDf2 = spark.read.parquet("employees.parquet")
empoyeeDf2.show
```

There's no need to include any extra packages when launching ```spark-shell``` as was the case with Avro.
