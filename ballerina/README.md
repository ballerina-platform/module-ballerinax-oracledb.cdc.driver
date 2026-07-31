## Overview

The Oracle Change Data Capture (CDC) driver enables capturing and tracking data changes in Oracle databases in real-time. It provides a reliable way to stream data modifications (Insert, Update, Delete) to downstream systems for processing or synchronization. The CDC driver is essential for building real-time data integration and synchronization pipelines.

### Key Features

- Real-time capturing of data changes (Insert, Update, Delete)
- Seamless streaming of data modifications to downstream systems
- Reliable tracking of database changes with minimal overhead
- Support for the LogMiner-based CDC flow and its configuration
- Secure communication and efficient data handling
- GraalVM compatible for native image builds


This library provides the necessary Debezium drivers required for the CDC (Change Data Capture) connector in Ballerina.
It enables listening to changes in Oracle databases seamlessly within Ballerina projects.

## Compatibility

| |     Version     |
|:---|:---------------:|
|Ballerina Language |  **2201.12.0**   |
|Debezium Oracle Driver | **3.5.1.Final** |

## Usage

To include the `oracledb.cdc.driver` dependency in your project, simply import the module as shown below:

```ballerina
import ballerinax/cdc;
import ballerinax/oracledb.cdc.driver as _;
```

The `oracledb.cdc.driver` library is bundled with the required Debezium Oracle connector JAR, and transitively pulls in the Oracle JDBC driver via `ballerinax/oracledb.driver`. No additional configuration is needed for the common LogMiner-based CDC flow.
