## Overview

[Oracle XStream](https://docs.oracle.com/database/121/XSTRM/xstrm_intro.htm#XSTRM72647) is a mechanism for applications 
to receive changes from an Oracle database in real time. This functionality is very similar to [Oracle GoldenGate](http://www.oracle.com/technetwork/middleware/goldengate/overview/index.html),
enough so that it requires a GoldenGate license. 

## Key Priorities

The connector will use the primary key of the table for they key in Kafka. If there is no primary key it will use the 
first unique constraint for the table. If there are no unique contraints for the table the ROWID of the row will be used
as the key. This will ensure that operations will end up in the same partition in Kafka.


## Installation

The Oracle XStream API requires the Oracle OCI (Thick) Client. Download the proper client from 
[Oracle Instant Client downloads](http://www.oracle.com/technetwork/topics/intel-macsoft-096467.html).

1. Download [instantclient-basic-macos.x64-12.1.0.2.0.zip](http://www.oracle.com/technetwork/topics/intel-macsoft-096467.html)
1. Extract contents to `oracle/instantclient_11_2`
1. Download [instantclient-basic-macos.x64-11.2.0.4.0.zip](http://www.oracle.com/technetwork/topics/intel-macsoft-096467.html)
1. Extract contents to `instantclient_12_1`

### Install artifacts in local repository

#### Oracle 11g

```
mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.4 -Dpackaging=jar -Dfile=oracle/instantclient_11_2/ojdbc6.jar
mvn install:install-file -DgroupId=com.oracle -DartifactId=xstreams -Dversion=11.2.0.4 -Dpackaging=jar -Dfile=oracle/instantclient_11_2/xstreams.jar
```

#### Oracle 12c

```
mvn install:install-file  -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=12.1.0.2 -Dpackaging=jar -Dfile=oracle/instantclient_12_1/ojdbc6.jar
mvn install:install-file  -DgroupId=com.oracle -DartifactId=xstreams -Dversion=12.1.0.2 -Dpackaging=jar -Dfile=oracle/instantclient_12_1/xstreams.jar
```

### Upload artifacts to Nexus

#### Oracle 11g

```
export NEXUS_URL='https://nexus.example.com/repository/maven-releases/'
export NEXUS_REPO_ID='ldap-jeremy'
mvn deploy:deploy-file -DrepositoryId=$NEXUS_REPO_ID -Durl=$NEXUS_URL -DgeneratePom=true -Dpackaging=jar -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.4 -Dfile=oracle/instantclient_11_2/ojdbc6.jar
mvn deploy:deploy-file -DrepositoryId=$NEXUS_REPO_ID -Durl=$NEXUS_URL -DgeneratePom=true -Dpackaging=jar -DgroupId=com.oracle -DartifactId=xstreams -Dversion=11.2.0.4 -Dfile=oracle/instantclient_11_2/xstreams.jar
```


#### Oracle 12c

```
export NEXUS_URL='https://nexus.example.com/repository/maven-releases/'
export NEXUS_REPO_ID='ldap-jeremy'
mvn deploy:deploy-file -DrepositoryId=$NEXUS_REPO_ID -Durl=$NEXUS_URL -DgeneratePom=true -Dpackaging=jar -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=12.1.0.2 -Dfile=oracle/instantclient_12_1/ojdbc6.jar
mvn deploy:deploy-file -DrepositoryId=$NEXUS_REPO_ID -Durl=$NEXUS_URL -DgeneratePom=true -Dpackaging=jar -DgroupId=com.oracle -DartifactId=xstreams -Dversion=12.1.0.2 -Dfile=oracle/instantclient_12_1/xstreams.jar
```

### Configure docker images

Oracle makes things nice and difficult by not offering any binary images of the Oracle Database. 

#### Oracle 12c

1. Clone the official [Oracle Docker Images](https://github.com/oracle/docker-images).
1. Follow the [instructions](https://github.com/oracle/docker-images/tree/master/OracleDatabase#how-to-build-and-run) to build.
1. 

#### Oracle 11g

# Configuration

## OracleSourceConnector

The OracleSourceConnector leverages the [Oracle XStream](https://docs.oracle.com/database/121/XSTRM/xstrm_intro.htm#XSTRM72647) API to stream changes from an Oracle instance in real time.

```properties
name=connector1
tasks.max=1
connector.class=com.github.jcustenborder.kafka.connect.cdc.xstream.OracleSourceConnector

# Set these required values
initial.database=
server.name=
password=
server.port=
oracle.xstream.server.names=
username=
```

| Name                            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Type     | Default                          | Valid Values                                                                                                                         | Importance |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|------------|
| initial.database                | The initial database to connect to.                                                                                                                                                                                                                                                                                                                                                                                                                                   | string   |                                  |                                                                                                                                      | high       |
| oracle.xstream.server.names     | Name of the XStream outbound servers.                                                                                                                                                                                                                                                                                                                                                                                                                                 | list     |                                  |                                                                                                                                      | high       |
| password                        | JDBC Password to connect to the database with.                                                                                                                                                                                                                                                                                                                                                                                                                        | password |                                  |                                                                                                                                      | high       |
| server.name                     | The server to connect to.                                                                                                                                                                                                                                                                                                                                                                                                                                             | string   |                                  |                                                                                                                                      | high       |
| server.port                     | The port on the server to connect to.                                                                                                                                                                                                                                                                                                                                                                                                                                 | int      |                                  |                                                                                                                                      | high       |
| username                        | JDBC Username to connect to the database with.                                                                                                                                                                                                                                                                                                                                                                                                                        | string   |                                  |                                                                                                                                      | high       |
| schema.key.name.format          | Format used to generate the name for the key schema. The following template properties are available for string replacement. `${databaseName}`, `${schemaName}`, `${tableName}`, `${namespace}`                                                                                                                                                                                                                                                                       | string   | ${namespace}.${tableName}Key     |                                                                                                                                      | high       |
| schema.namespace.format         | The namespace for the schemas generated by the connector. The following template properties are available for string replacement. `${databaseName}`, `${schemaName}`, `${tableName}`, `${namespace}`                                                                                                                                                                                                                                                                  | string   | com.example.data.${databaseName} |                                                                                                                                      | high       |
| schema.value.name.format        | Format used to generate the name for the value schema. The following template properties are available for string replacement. `${databaseName}`, `${schemaName}`, `${tableName}`, `${namespace}`                                                                                                                                                                                                                                                                     | string   | ${namespace}.${tableName}Value   |                                                                                                                                      | high       |
| topicFormat.format              | The topicFormat to write the data to.                                                                                                                                                                                                                                                                                                                                                                                                                                 | string   | ${databaseName}.${tableName}     |                                                                                                                                      | high       |
| jdbc.pool.max.idle              | The maximum number of idle CONNECTIONS in the connection pool.                                                                                                                                                                                                                                                                                                                                                                                                        | int      | 10                               |                                                                                                                                      | medium     |
| jdbc.pool.max.total             | The maximum number of CONNECTIONS for the connection pool to open. If a number greater than this value is requested, the caller will block waiting for a connection to be returned.                                                                                                                                                                                                                                                                                   | int      | 30                               |                                                                                                                                      | medium     |
| jdbc.pool.min.idle              | The minimum number of idle CONNECTIONS in the connection pool.                                                                                                                                                                                                                                                                                                                                                                                                        | int      | 3                                |                                                                                                                                      | medium     |
| oracle.xstream.batch.interval   | XStreamOut batch processing interval.                                                                                                                                                                                                                                                                                                                                                                                                                                 | int      | 30                               |                                                                                                                                      | medium     |
| oracle.xstream.idle.timeout     | XStreamOut idle timeout value.                                                                                                                                                                                                                                                                                                                                                                                                                                        | int      | 1                                |                                                                                                                                      | medium     |
| backoff.time.ms                 | The number of milliseconds to wait when no records are returned.                                                                                                                                                                                                                                                                                                                                                                                                      | int      | 1000                             | [50,...]                                                                                                                             | low        |
| batch.size                      | The number of records to return in a batch.                                                                                                                                                                                                                                                                                                                                                                                                                           | int      | 512                              | [1,...]                                                                                                                              | low        |
| oracle.xstream.receive.wait.ms  | The amount of time to wait in milliseconds when XStreamOut.receiveChange() returns null                                                                                                                                                                                                                                                                                                                                                                               | int      | 1000                             |                                                                                                                                      | low        |
| schema.cache.ms                 | The number of milliseconds to cache schema metadata in memory.                                                                                                                                                                                                                                                                                                                                                                                                        | int      | 300000                           | [60000,...]                                                                                                                          | low        |
| schema.caseformat.column.name   | This setting is used to control how the column names are cased when the resulting schemas are generated.                                                                                                                                                                                                                                                                                                                                                              | string   | NONE                             | ValidEnum{enum=CaseFormat, allowed=[LOWER_HYPHEN, LOWER_UNDERSCORE, LOWER_CAMEL, LOWER, UPPER_CAMEL, UPPER_UNDERSCORE, UPPER, NONE]} | low        |
| schema.caseformat.database.name | This setting is used to control how the `${databaseName}` variable is cased when it is passed to the formatters defined in the `schema.namespace.format`, `schema.key.name.format`, `schema.value.name.format`, `topicFormat.format` settings. This allows you to control the naming applied to these properties. For example this can be used to take a database name of `USER_TRACKING` to a more java like case of `userTracking` or all lowercase `usertracking`. | string   | NONE                             | ValidEnum{enum=CaseFormat, allowed=[LOWER_HYPHEN, LOWER_UNDERSCORE, LOWER_CAMEL, LOWER, UPPER_CAMEL, UPPER_UNDERSCORE, UPPER, NONE]} | low        |
| schema.caseformat.input         | The naming convention used by the database format. This is used to define the source naming convention used by the other `schema.caseformat.*` properties.                                                                                                                                                                                                                                                                                                            | string   | UPPER_UNDERSCORE                 | ValidEnum{enum=CaseFormat, allowed=[LOWER_HYPHEN, LOWER_UNDERSCORE, LOWER_CAMEL, UPPER_CAMEL, UPPER_UNDERSCORE]}                     | low        |
| schema.caseformat.schema.name   | This setting is used to control how the `${schemaName}` variable is cased when it is passed to the formatters defined in the `schema.namespace.format`, `schema.key.name.format`, `schema.value.name.format`, `topicFormat.format` settings. This allows you to control the naming applied to these properties. For example this can be used to take a schema name of `SCOTT` to a more java like case of `Scott` or all lowercase `scott`.                           | string   | NONE                             | ValidEnum{enum=CaseFormat, allowed=[LOWER_HYPHEN, LOWER_UNDERSCORE, LOWER_CAMEL, LOWER, UPPER_CAMEL, UPPER_UNDERSCORE, UPPER, NONE]} | low        |
| schema.caseformat.table.name    | This setting is used to control how the `${tableName}` variable is cased when it is passed to the formatters defined in the `schema.namespace.format`, `schema.key.name.format`, `schema.value.name.format`, `topicFormat.format` settings. This allows you to control the naming applied to these properties. For example this can be used to take a table name of `USER_SETTING` to a more java like case of `UserSetting` or all lowercase `usersetting`.          | string   | NONE                             | ValidEnum{enum=CaseFormat, allowed=[LOWER_HYPHEN, LOWER_UNDERSCORE, LOWER_CAMEL, LOWER, UPPER_CAMEL, UPPER_UNDERSCORE, UPPER, NONE]} | low        |
| xstream.allowed.commands        | The commands the task should process.                                                                                                                                                                                                                                                                                                                                                                                                                                 | list     | [INSERT, UPDATE, DELETE]         |                                                                                                                                      | low        |

