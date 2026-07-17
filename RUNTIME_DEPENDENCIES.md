# Oracle CDC Driver — Runtime Dependency Reference

This document tracks the full set of JARs that the Debezium Oracle connector
(`io.debezium:debezium-connector-oracle:3.5.1.Final`) can require at runtime, and
records **which ones this driver bundles** versus which are expected to come from
elsewhere on the classpath.

It exists because `ballerina/Ballerina.toml` deliberately bundles only a small subset
of the Oracle connector's dependencies. If a consumer hits a runtime
`ClassNotFoundException` or `NoClassDefFoundError`, this table is the first place to
look: the missing class almost certainly belongs to one of the JARs listed under
"Not bundled" below.

> Source: the `<dependencies>` block of the published
> [`debezium-connector-oracle:3.5.1.Final` POM](https://repo1.maven.org/maven2/io/debezium/debezium-connector-oracle/3.5.1.Final/)
> and the [Debezium Oracle connector docs](https://debezium.io/documentation/reference/stable/connectors/oracle.html).
> Versions marked _"(BOM-managed)"_ are pinned transitively by the Debezium parent
> BOM and have no fixed value here — resolve them from the dependency tree when adding.

## Currently bundled (in `Ballerina.toml`)

| Group | Artifact | Version | Notes |
|---|---|---|---|
| `io.debezium` | `debezium-connector-oracle` | `3.5.1.Final` | The Oracle LogMiner connector. |
| `io.debezium` | `debezium-ddl-parser` | `3.5.1.Final` | DDL statement parsing during snapshot. Package `io.debezium.ddl.parser.*`, `io.debezium.antlr.*`. |
| `org.antlr` | `antlr4-runtime` | `4.10.1` | Required by `debezium-ddl-parser`. Package `org.antlr.v4.runtime.*`. (Same version mysql.cdc.driver bundles.) |
| `io.ballerina.lib` | `oracledb.cdc.driver-native` | `1.0.0` | This module's native-image marker JAR (GraalVM reflect/resource config). Built locally. |

## Provided elsewhere (do NOT bundle)

| Group | Artifact | Provided by | Notes |
|---|---|---|---|
| `com.oracle.database.jdbc` | `ojdbc11` | `ballerinax/oracledb.driver` (transitive via `init.bal`) | Oracle JDBC driver. Already on the classpath whenever `oracledb.cdc.driver` is imported. Pinned to `23.2.0.0` by the `oracledb` connector. |
| `io.debezium` | `debezium-core` | `ballerinax/cdc` | Core Debezium engine. Shipped by the core CDC module. |
| `io.debezium` | `debezium-api` | `ballerinax/cdc` | Debezium public API. |
| `io.debezium` | `debezium-embedded` | `ballerinax/cdc` | Embedded engine used by the Ballerina CDC runtime. |
| `io.debezium` | `debezium-storage-file` | `ballerinax/cdc` | Offset/history file storage backend. |
| `io.debezium` | `debezium-storage-kafka` | `ballerinax/cdc` | Kafka storage backend. |
| `org.apache.kafka` | `connect-api` | `ballerinax/cdc` | Kafka Connect API (`provided` scope in the Debezium POM). |
| `org.slf4j` | `slf4j-api` | `ballerinax/cdc` / Ballerina runtime | Logging facade (`provided` scope). |

## Not bundled — add here only if a runtime error proves it is missing

These are runtime dependencies of `debezium-connector-oracle` that are **not** known
to be provided by `ballerinax/cdc`. They are intentionally omitted to keep the driver
lean. If a `ClassNotFoundException` points at one of these packages, add the JAR as a
new `[[platform.java21.dependency]]` in `build-config/resources/Ballerina.toml`, add a
download entry in the root `build.gradle` `externalJars` configuration, and re-pin the
version below.

| Group | Artifact | Version | Triggering feature / package |
|---|---|---|---|
| `com.oracle.database.nls` | `orai18n` | (BOM-managed) | Oracle character-set / i18n handling for non-ASCII data. Package `oracle.i18n.*`. |
| `org.infinispan` | `infinispan-core` | (BOM-managed) | Optional cache backend for the LogMiner transaction buffer (`log.mining.buffer.type=infinispan_embedded`). Package `org.infinispan.*`. |
| `org.infinispan` | `infinispan-client-hotrod` | (BOM-managed) | Optional remote cache for the LogMiner transaction buffer (`log.mining.buffer.type=infinispan_remote`). |
| `org.infinispan.protostream` | `protostream-processor` | (BOM-managed) | Protobuf serialisation used by the Infinispan buffer. |
| `com.google.protobuf` | `protobuf-java` | (BOM-managed) | Used by Infinispan protostream serialisation. Package `com.google.protobuf.*`. |
| `com.google.protobuf` | `protobuf-java-util` | (BOM-managed) | Protobuf utilities. |
| `org.ehcache` | `ehcache` | (BOM-managed) | Optional cache backend for the LogMiner transaction buffer (`log.mining.buffer.type=ehcache`). Package `org.ehcache.*`. |
| `org.glassfish.jaxb` | `jaxb-runtime` | (BOM-managed) | JAXB implementation for XML binding. Package `org.glassfish.jaxb.*`. |
| `com.oracle.database.xml` | `xdb` | (BOM-managed) | XMLType column support. `provided` scope in the Debezium POM — only needed if capturing `XMLTYPE` columns. Package `oracle.xdb.*`. |
| `com.oracle.database.xml` | `xmlparserv2` | (BOM-managed) | XML parsing, partner to `xdb`. `provided` scope. Package `oracle.xml.*`. |

## Explicitly out of scope

| Group | Artifact | Reason |
|---|---|---|
| `com.oracle.instantclient` | `xstreams` | Only required for the Oracle **XStream** adapter. The Ballerina Oracle CDC implementation supports the **LogMiner** adapter only, so XStream and its `xstreams.jar` (which is not freely redistributable) are never needed. |

## How to add a JAR if needed

1. Add the dependency to the `externalJars` configuration in the root
   [`build.gradle`](./build.gradle) so the Gradle `copyToLib` task downloads it into
   `ballerina/lib/`:

   ```gradle
   externalJars(group: '<group>', name: '<artifact>', version: "<version>") {
       transitive = false
   }
   ```

2. Add a matching `[[platform.java21.dependency]]` block to the template
   [`build-config/resources/Ballerina.toml`](./build-config/resources/Ballerina.toml)
   (the generated `ballerina/Ballerina.toml` is overwritten from this template on every build):

   ```toml
   [[platform.java21.dependency]]
   groupId = "<group>"
   artifactId = "<artifact>"
   version = "<version>"
   path = "./lib/<artifact>-<version>.jar"
   ```

3. If the JAR contains Debezium connector classes instantiated reflectively, add the
   relevant entries to
   [`native/.../oracledb.cdc.driver-native/reflect-config.json`](./native/src/main/resources/META-INF/native-image/io.ballerina.lib/oracledb.cdc.driver-native/reflect-config.json)
   so GraalVM native builds keep working.

4. Re-run `./gradlew clean build` and confirm the JAR is packed into the generated
   `.bala` (`unzip -l ballerina/target/bala/*.bala | grep <artifact>`).
