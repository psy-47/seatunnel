# Intro To Connector V2 Features

## Differences Between Connector V2 And V1

Since https://github.com/apache/seatunnel/issues/1608 We Added Connector V2 Features.
Connector V2 is a connector defined based on the SeaTunnel Connector API interface. Unlike Connector V1, V2 supports the following features:

* **Multi Engine Support** SeaTunnel Connector API is an engine independent API. The connectors developed based on this API can run in multiple engines. Currently, Flink and Spark are supported, and we will support other engines in the future.
* **Multi Engine Version Support** Decoupling the connector from the engine through the translation layer solves the problem that most connectors need to modify the code in order to support a new version of the underlying engine.
* **Unified Batch And Stream** Connector V2 can perform batch processing or streaming processing. We do not need to develop connectors for batch and stream separately.
* **Multiplexing JDBC/Log connection.** Connector V2 supports JDBC resource reuse and sharing database log parsing.

## Source Connector Features

Source connectors have some common core features, and each source connector supports them to varying degrees.

### exactly-once

If each piece of data in the data source will only be sent downstream by the source once, we think this source connector supports exactly once.

In SeaTunnel, we can save the read **Split** and its **offset** (The position of the read data in split at that time,
such as line number, byte size, offset, etc.) as **StateSnapshot** when checkpointing. If the task restarted, we will get the last **StateSnapshot**
and then locate the **Split** and **offset** read last time and continue to send data downstream.

For example `File`, `Kafka`.

### column projection

If the connector supports reading only specified columns from the data source (Note that if you read all columns first and then filter unnecessary columns through the schema, this method is not a real column projection)

For example `JDBCSource` can use sql to define reading columns.

`KafkaSource` will read all content from topic and then use `schema` to filter unnecessary columns, This is not `column projection`.

### batch

Batch Job Mode, The data read is bounded and the job will stop after completing all data read.

### stream

Streaming Job Mode, The data read is unbounded and the job never stop.

### parallelism

Parallelism Source Connector support config `parallelism`, every parallelism will create a task to read the data.
In the **Parallelism Source Connector**, the source will be split into multiple splits, and then the enumerator will allocate the splits to the SourceReader for processing.

### support user-defined split

User can config the split rule.

### support multiple table read

Supports reading multiple tables in one SeaTunnel job

## Sink Connector Features

Sink connectors have some common core features, and each sink connector supports them to varying degrees.

### exactly-once

When any piece of data flows into a distributed system, if the system processes any piece of data accurately only once in the whole processing process and the processing results are correct, it is considered that the system meets the exact once consistency.

For sink connector, the sink connector supports exactly-once if any piece of data only write into target once. There are generally two ways to achieve this:

* The target database supports key deduplication. For example `MySQL`, `Kudu`.
* The target support **XA Transaction**(This transaction can be used across sessions. Even if the program that created the transaction has ended, the newly started program only needs to know the ID of the last transaction to resubmit or roll back the transaction). Then we can use **Two-phase Commit** to ensure **exactly-once**. For example `File`, `MySQL`.

### cdc(change data capture)

If a sink connector supports writing row kinds(INSERT/UPDATE_BEFORE/UPDATE_AFTER/DELETE) based on primary key, we think it supports cdc(change data capture).
