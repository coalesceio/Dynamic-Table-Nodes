# Dynamic Tables Package

Package includes:

* [Dynamic Table Work](#dynamic-table-work)
* [Dynamic Table Dimension](#dynamic-table-dimension)
* [Dynamic Table Latest Record Version](#dynamic-table-latest-record-version)

---

## Dynamic Table Work

The Coalesce Dynamic Table Work UDN is a versatile node that allows you to develop and deploy a single Dynamic Table Work or a DAG of Dynamic Tables in Snowflake.

[Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about) are a new table type offered by Snowflake that allow data teams to use SQL statements to declaratively define the results of data pipelines. Dynamic tables simplify the process of creating and managing data pipelines by streamlining data transformations without having to manage Streams and Tasks. 

### Dynamic Table Work Node Configuration

The Dynamic Table Work has three configuration groups:

* [Node Properties](#dynamic-table-work-node-properties)
* [Dynamic Table Options](#dynamic-table-work-options) 
* [General Options](#dynamic-table-work-general-options)
* [Advanced Options](#dynamic-table-work-advanced-options)

#### Dynamic Table Work Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | (Required) Storage Location where the Dynamic Table will be created |
| **Node Type** | (Required) Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed/redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### Dynamic Table Work Options

![dynamic table options](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/169126315/e4fbc362-f0c6-463c-9e82-ac454709c700)

| **Option** | **Description** |
|------------|----------------|
| **Warehouse** | (Required) Name of warehouse used to refresh the Dynamic Table |
| **Downstream** | (Required) True/False toggle:<br/>- **True**: Refresh on demand when dependent tables need refresh<br/>- **False**: Set Lag Specification for refresh schedule |
| **Lag Specification** | Only if Downstream is False. Review [Snowflakes Dynamic Tables Refresh](https://docs.snowflake.com/en/user-guide/dynamic-tables-refresh) to understand how to specify the target lag. Set refresh schedule with:<br/>- **Time Value**: Frequency of the refresh<br/>- **Time Period**: Seconds/Minutes/Hours/Days |
| **Refresh_Mode** | (Required) Specifies refresh type:<br/>- **AUTO**: Default incremental refresh. If the CREATE DYNAMIC TABLE statement does not support the incremental refresh mode, the dynamic table is automatically created with the full refresh mode.<br/>- **INCREMENTAL**: Force incremental refresh<br/>- **FULL**: Force full refresh |
| **Initialize** | (Required) Initial refresh behavior:<br/>- **ON_CREATE**: Refresh synchronously at creation<br/>- **ON_SCHEDULE**: Refresh at next scheduled time |

#### Dynamic Table Work Iceberg Options

| **Option** | **Description** |
|------------|----------------|
| **Snowflake EXTERNAL VOLUME name** | Specifies the identifier (name) for the external volume where the Iceberg table stores its metadata files and data in Parquet format. [External volume](https://docs.snowflake.com/sql-reference/sql/create-external-volume) needs to be created in snowflake as a prerequisite. |
| **Base location name** | The path to a directory where Snowflake can write data and metadata files for the table. Specify a relative path from the table's EXTERNAL_VOLUME location. |

#### Dynamic Table Work General Options

![dynamic table1 1_general](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/7216836/2c663f13-8e7c-43c0-a7e0-c4abc1edfa14)

| **Option** | **Description** |
|------------|----------------|
| **Distinct** | True/False toggle to return DISTINCT rows |
| **Group By All** | True/False toggle to add non-aggregated columns to GROUP BY |
| **Multi Source** | True/False toggle for combining multiple sources via UNION or UNION ALL |
| **Create As** | Choose 'dynamic table' or 'transient dynamic table' |
| **Cluster key** | True/False toggle for clustering:<br/>- **True**: Specify clustering column and optional expressions<br/>- **False**: No clustering |
| **Allow Expressions Cluster Key**| When cluster key is set to true. Allows to add an expression to the specified cluster key| 

#### Dynamic Table Work Advanced Options

| **Option** | **Description** |
|------------|----------------|
| **Copy grants** | Specifies to retain the access privileges from the original table when a new dynamic table is created.Useful during replication.[More info on replication here](https://docs.snowflake.com/en/user-guide/account-replication-considerations#replication-and-dynamic-tables) |

### Dynamic Table Work Deployment

#### Dynamic Table Work Initial Deployment Parameters

The Dynamic Table Work includes an environment parameter that allows you to specify a different warehouse to refresh a Dynamic Table in different environments.

The parameter name is `targetDynamicTableWarehouse` and the default value is `DEV ENVIRONMENT`.

When set to `DEV ENVIRONMENT`, the value entered in the Dynamic Table Options config "Warehouse on which to execute Dynamic Table" will be used when creating the Dynamic Table.

```json
{
    "targetDynamicTableWarehouse": "DEV ENVIRONMENT"
}
```

When set to any value other than `DEV ENVIRONMENT` the node will attempt to create the Dynamic Table using a Snowflake warehouse with the specified value.

For example, the Dynamic Table will refresh using a warehouse named `compute_wh`.

```json
{
    "targetDynamicTableWarehouse": "compute_wh"
}
```
> 📘 **Deployment of nodes without adding parameters**
>
> This results in a WARNING stage getting executed insisting to execute the node after adding parameters 

### Dynamic Table Work Initial Deployment

When deployed for the first time into an environment the Dynamic Table Work node will execute the following stage:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Work Dynamic Table/Dynamic Transient Table** | This stage will execute a `CREATE OR REPLACE` statement and create a Dynamic Table in the target environment. |

### Deploying a DAG of Dynamic Tables

When a DAG of related Dynamic Tables are deployed together Coalesce will deploy the Dynamic Tables in the order that the Dynamic Tables are ordered.

#### Dynamic Table Work Redeployment

After initial deployment, subsequent deployments may alter or recreate the Dynamic Table.

#### Altering the Dynamic Table Work

The following config changes trigger ALTER statements:

1. Warehouse name
2. Downstream setting  
3. Lag specification

These execute the two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Alter Dynamic Table** | Executes ALTER to modify parameters |
| **Refresh Dynamic Table** | Refreshes table to make data available |

Also if the location of the node, node name, column level description, and table level description results in an `ALTER` statement, whereas other column or table level changes like data type change, column name change, column addition/deletion result in a `CREATE` statement.

### Changing Materialization Type From Dynamic Table to Transient Dynamic Table

If the materialization type changes from dynamic table to transient dynamic table and there are changes in dynamic table config options or table level changes that result in ALTER, the following steps are executed:

1. **Clone Work node**
2. **Swap Work node**
3. **Drop dynamic table**
4. **Alter Dynamic Table**
5. **Refresh Dynamic Table**
6. **Apply Table Clustering(if cluster key option is provided)**
7. **Resume Recluster Table(if cluster key option is provided)**

If the materialization type changes from dynamic table to transient dynamic table and there are changes in dynamic table config options or table level changes that result in `CREATE`, the following steps are executed:

1. **Drop dynamic table**
2. **Create Work dynamic transient table**
3. **Apply Table Clustering(if cluster key option is provided)**
4. **Resume Recluster Table(if cluster key option is provided)**

### Changing Materialization Type From Transient Dynamic Table to Dynamic Table

If the materialization type from transient dynamic table to dynamic table and there are changes in dynamic table config options, the following steps gets executed:

1. **Drop transient dynamic table**
2. **Create Work dynamic table**
3. **Apply Table Clustering(if cluster key option is provided)**
4. **Resume Recluster Table(if cluster key option is provided)**

### Recreating the Dynamic Table

If anything changes other than the configuration options specified in [Altering the Dynamic Table](#altering-the-dynamic-table) then the Dynamic Table will be recreated by running a `CREATE OR REPLACE `statement.

If the changes in node results in recreating the Dynamic table,then following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Drop table/transient table** | Table is dropped before recreating in case the node name or location is changed |
| **Create Work Dynamic table/Dynamic transient table**| Dynamic table is created|

### Redeploying a DAG of Dynamic Tables Work

If an entire DAG of Dynamic Tables has been deployed and changes are made to a deployed Dynamic Table Coalesce will only redeploy Dynamic Tables that have changed metadata.

### Redeployment with no changes 

If the nodes are redeployed with no changes compared to previous deployment,then no stages are executed

### Dynamic Tables Work Undeployment

A table will be dropped if all of these are true:

* The Dynamic Work Node is deleted from a Workspace.
* The Workspace is committed to Git.
* The Workspace committed to Git is deployed to a higher level environment.

| **Stage** | **Description** |
|-----------|----------------|
| **Drop Dynamic Table** | Removes table from target environment |

---

## Dynamic Table Dimension

The Coalesce Dynamic Table Dimension UDN is a versatile node that allows you to develop and deploy a single Dynamic Table Dimension or a DAG of Dynamic Tables in Snowflake.

[Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about) are a new table type offered by Snowflake that allow data teams to use SQL statements to declaratively define the results of data pipelines. Dynamic tables simplify the process of creating and managing data pipelines by streamlining data transformations without having to manage Streams and Tasks.

### Dimension Node Configuration

The Dynamic Table Dimension has four configuration groups:

* [Node Properties](#dimension-node-properties)
* [Dynamic Table Options](#dimension-table-options)
* [Dimension Options](#dimension-options)
* [General Options](#dimension-general-options)
* [Advanced Options](#dimension-advanced-options)

#### Dimension Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where table will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed/redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### Dimension Table Options

![dynamic table options](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/169126315/44235450-1383-4530-a68a-28f040bd5c48)

| **Option** | **Description** |
|------------|----------------|
| **Warehouse** | (Required) Name of warehouse used to refresh the Dynamic Table |
| **Downstream** | (Required) True/False toggle:<br/>- **True**: Refresh on demand when dependent tables need refresh<br/>- **False**: Set Lag Specification for refresh schedule |
| **Lag Specification** | Only if Downstream is False. Review [Snowflakes Dynamic Tables Refresh](https://docs.snowflake.com/en/user-guide/dynamic-tables-refresh) to understand how to specify the target lag. Set refresh schedule with:<br/>- **Time Value**: Frequency of refresh for a given Time Period.<br/>- **Time Period**: Seconds/Minutes/Hours/Days |
| **Refresh_Mode** | (Required) Specifies refresh type:<br/>- **AUTO**: Default incremental refresh. If the CREATE DYNAMIC TABLE statement does not support the incremental refresh mode, the dynamic table is automatically created with the full refresh mode.<br/>- **INCREMENTAL**: Force incremental refresh<br/>- **FULL**: Force full refresh |
| **Initialize** | (Required) Initial refresh behavior:<br/>- **ON_CREATE**: Refresh synchronously at creation<br/>- **ON_SCHEDULE**: Refresh at next scheduled time |

#### Dimension Options

![Dimension options](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/169126315/b3f8d788-b8a8-4884-b9bd-2ff85039d9df)

| **Option** | **Description** |
|------------|----------------|
| **Table keys** | (Required) Business key columns for Dimension key formation |
| **Record versioning** | (Required) Type of column for history maintenance:<br/>- Datetime column<br/>- Date and Time column<br/>- Integer column |
| **Timestamp** | Required if Datetime column chosen for Record versioning.**Note:If multiple columns are chosen.The first timestamp column chosen is considered for versioning order** |
| **Sequence** | Required if Integer column chosen for Record versioning|
| **Timetamp-track data load**| Required if Integer column chosen for Record versioning**Note:If multiple columns are chosen.The first timestamp column chosen is considered for versioning order** |
| **Date/Timestamp Columns** | Required if Date and Time columns chosen for Record versioning**Note:If multiple columns are chosen.The first timestamp column chosen is considered for versioning order** |

#### Dimension Iceberg Options

| **Option** | **Description** |
|------------|----------------|
| **Snowflake EXTERNAL VOLUME name** | Specifies the identifier (name) for the external volume where the Iceberg table stores its metadata files and data in Parquet format. [External volume](https://docs.snowflake.com/sql-reference/sql/create-external-volume) needs to be created in snowflake as a prerequisite. |
| **Base location name** | The path to a directory where Snowflake can write data and metadata files for the table. Specify a relative path from the table's EXTERNAL_VOLUME location. |

#### Dimension General Options

![dynamictable1 1-dimgeneral](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/7216836/dda1c53c-a3da-4701-8449-0023db1ac44a)

| **Option** | **Description** |
|------------|----------------|
| **Create As** | Choose 'dynamic table' or 'transient dynamic table' |
| **Cluster key** | True/False toggle for clustering:<br/>- **True**: Specify clustering column and optional expressions<br/>- **False**: No clustering |
| **Allow Expressions Cluster Key**| When cluster key is set to true. Allows to add an expression to the specified cluster key|

#### Dimension Advanced Options

| **Option** | **Description** |
|------------|----------------|
| **Copy grants** | Specifies to retain the access privileges from the original table when a new dynamic table is created.Useful during replication.[More info on replication here](https://docs.snowflake.com/en/user-guide/account-replication-considerations#replication-and-dynamic-tables) |

### Dimension Deployment

#### Dimension Initial Deployment Parameters

The Dynamic Table Work includes an environment parameter that allows you to specify a different warehouse to refresh a Dynamic Table in different environments.

The parameter name is `targetDynamicTableWarehouse` and the default value is `DEV ENVIRONMENT`.

When set to `DEV ENVIRONMENT`, the value entered in the Dynamic Table Options config "Warehouse on which to execute Dynamic Table" will be used when creating the Dynamic Table.

```json
{
    "targetDynamicTableWarehouse": "DEV ENVIRONMENT"
}
```

When set to any value other than `DEV ENVIRONMENT` the node will attempt to create the Dynamic Table using a Snowflake warehouse with the specified value.

For example, the Dynamic Table will refresh using a warehouse named `compute_wh`.

```json
{
    "targetDynamicTableWarehouse": "compute_wh"
}
```
> 📘 **Deployment of nodes without adding parameters**
>
> This results in a WARNING stage getting executed insisting to execute the node after adding parameters
 
### Dimension Initial Deployment

When deployed for the first time into an environment the Dynamic Table Work node will execute the following stage:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Dimension Dynamic Table/Dynamic Transient Table** | This stage will execute a `CREATE OR REPLACE` statement and create a Dynamic Table in the target environment. |

### Deploying a DAG of Dimension Tables

When a DAG of related Dynamic Tables are deployed together Coalesce will deploy the Dynamic Tables in the order that the Dynamic Tables are ordered.

#### Dimension Redeployment

After initial deployment, subsequent deployments may alter or recreate the Dynamic Table.

#### Altering the Dimension Table

The following config changes trigger ALTER statements:

1. Warehouse name
2. Downstream setting  
3. Lag specification

These execute the two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Alter Dynamic Table** | Executes ALTER to modify parameters |
| **Refresh Dynamic Table** | Refreshes table to make data available |

Also if the location of the node, node name, column level description, and table level description results in an `ALTER` statement, whereas other column or table level changes like data type change, column name change, column addition/deletion result in a `CREATE` statement.

### Changing Materialization Type From Dimension Table to Transient Dimension Table

If the materialization type changes from dynamic table to transient dynamic table and there are changes in dynamic table config options or table level changes that result in ALTER, the following steps are executed:

1. **Clone Dimension node**
2. **Swap Dimension node**
3. **Drop Dynamic table**
4. **Alter Dynamic Table**
5. **Refresh Dynamic Table**
6. **Apply Table Clustering(if cluster key option is provided)**
7. **Resume Recluster Table(if cluster key option is provided)**

If the materialization type changes from dynamic table to transient dynamic table and there are changes in dynamic table config options or table level changes that result in `CREATE`, the following steps are executed:

1. **Drop dynamic table**
2. **Create dimension dynamic transient table**
3. **Apply Table Clustering(if cluster key option is provided)**
4. **Resume Recluster Table(if cluster key option is provided)**

### Changing Materialization Type From Transient Dynamic Dimension Table to Dynamic Table

If the materialization type from transient dynamic table to dynamic table and there are changes in dynamic table config options, the following steps gets executed:

1. **Drop transient dimension table**
2. **Create Work dimension table**
3. **Apply Table Clustering(if cluster key option is provided)**
4. **Resume Recluster Table(if cluster key option is provided)**

### Recreating the Dimension Table

If anything changes other than the configuration options specified in [Altering the Dimension Table](#altering-the-dimension-table) then the Dynamic Table will be recreated by running a `CREATE OR REPLACE`statement.

If the changes in node results in recreating the Dynamic table, then following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Drop table/transient table** | Table is dropped before recreating in case the node name or location is changed |
| **Create Work Dynamic table/Dynamic transient table**| Dynamic table is created|

### Redeploying a DAG of Dimension

If an entire DAG of Dynamic Tables has been deployed and changes are made to a deployed Dynamic Table Coalesce will only redeploy Dynamic Tables that have changed metadata.

### Redeployment with no changes 

If the nodes are redeployed with no changes compared to previous deployment,then no stages are executed

### Dimension Tables Work Undeployment

A table will be dropped if all of these are true:

* The Dynamic Dimension Node is deleted from a Workspace.
* The Workspace is committed to Git.
* The Workspace committed to Git is deployed to a higher level environment.

| **Stage** | **Description** |
|-----------|----------------|
| **Drop Dynamic Table** | Removes table from target environment |

---

## Dynamic Table Latest Record Version

The Coalesce Dynamic Table Latest Record Version UDN is a versatile node that allows you to develop and deploy a single Dynamic Table Work or a DAG of Dynamic Tables with only the latest version of rows in Snowflake.

[Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about. ) are a new table type offered by Snowflake that allow data teams to use SQL statements to declaratively define the results of data pipelines. Dynamic tables simplify the process of creating and managing data pipelines by streamlining data transformations without having to manage Streams and Tasks.

### Latest Record Version Node Configuration

The Dynamic Table Dimension has four configuration groups:

* [Node Properties](#latest-record-version-node-properties)
* [Dynamic Table Options](#latest-record-version-options)
* [Dimension Options](#latest-record-version-dimension-options)
* [General Options](#latest-record-version-general-options)
* [Advanced Options](#latest-record-version-advanced-options)

#### Latest Record Version Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where table will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed/redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### Latest Record Version Options

| **Option** | **Description** |
|------------|----------------|
| **Table keys** | (Required) Business key columns for Dimension key formation |
| **Record versioning** | (Required) Type of column for history maintenance:<br/>- Datetime column<br/>- Date and Time column<br/>- Integer column |
| **Timestamp** | Required if Datetime column chosen for Record versioning.**Note:If multiple columns are chosen.The first timestamp column chosen is considered for versioning order** |
| **Sequence** | Required if Integer column chosen for Record versioning|
| **Timetamp-track data load**| Required if Integer column chosen for Record versioning**Note:If multiple columns are chosen.The first timestamp column chosen is considered for versioning order** |
| **Date/Timestamp Columns** | Required if Date and Time columns chosen for Record versioning**Note:If multiple columns are chosen.The first timestamp column chosen is considered for versioning order** |


#### Latest Record Version Iceberg Options

| **Option** | **Description** |
|------------|----------------|
| **Snowflake EXTERNAL VOLUME name** | Specifies the identifier (name) for the external volume where the Iceberg table stores its metadata files and data in Parquet format. [External volume](https://docs.snowflake.com/sql-reference/sql/create-external-volume) needs to be created in snowflake as a prerequisite. |
| **Base location name** | The path to a directory where Snowflake can write data and metadata files for the table. Specify a relative path from the table's EXTERNAL_VOLUME location. |

#### Latest Record Version General Options

| **Option** | **Description** |
|------------|----------------|
| **Create As** | Choose 'dynamic table' or 'transient dynamic table' |
| **Cluster key** | True/False toggle for clustering:<br/>- **True**: Specify clustering column and optional expressions<br/>- **False**: No clustering |
| **Allow Expressions Cluster Key**| When cluster key is set to true. Allows to add an expression to the specified cluster key|

#### Latest Record Version Advanced Options

| **Option** | **Description** |
|------------|----------------|
| **Copy grants** | Specifies to retain the access privileges from the original table when a new dynamic table is created.Useful during replication.[More info on replication here](https://docs.snowflake.com/en/user-guide/account-replication-considerations#replication-and-dynamic-tables) |


### DAG of Dynamic Table Latest Record Version

When designing DAG of Dynamic tables, you should specify the target lag. Review [Understanding dynamic table refresh - Snowflake](https://docs.snowflake.com/en/user-guide/dynamic-tables-refresh)

### Latest Record Version Deployment

#### Latest Record Version Initial Deployment Parameters

The Dynamic Table Work includes an environment parameter that allows you to specify a different warehouse to refresh a Dynamic Table in different environments.

The parameter name is `targetDynamicTableWarehouse` and the default value is `DEV ENVIRONMENT`.

When set to `DEV ENVIRONMENT`, the value entered in the Dynamic Table Options config "Warehouse on which to execute Dynamic Table" will be used when creating the Dynamic Table.

```json
{
    "targetDynamicTableWarehouse": "DEV ENVIRONMENT"
}
```

When set to any value other than `DEV ENVIRONMENT` the node will attempt to create the Dynamic Table using a Snowflake warehouse with the specified value.

For example, the Dynamic Table will refresh using a warehouse named `compute_wh`.

```json
{
    "targetDynamicTableWarehouse": "compute_wh"
}
```
> 📘 **Deployment of nodes without adding parameters**
>
> This results in a WARNING stage getting executed insisting to execute the node after adding parameters
 
### Latest Record Version Initial Deployment

When deployed for the first time into an environment the Dynamic Table Work node will execute the following stage:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Dimension Dynamic Table/Dynamic Transient Table** | This stage will execute a `CREATE OR REPLACE` statement and create a Dynamic Table in the target environment. |

### Deploying a DAG of Latest Record Version Tables

When a DAG of related Dynamic Tables are deployed together Coalesce will deploy the Dynamic Tables in the order that the Dynamic Tables are ordered.

#### Latest Record Version Redeployment

After initial deployment, subsequent deployments may alter or recreate the Dynamic Table.

#### Altering the Latest Record Version Table

The following config changes trigger ALTER statements:

1. Warehouse name
2. Downstream setting  
3. Lag specification

These execute the two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Alter Dynamic Table** | Executes ALTER to modify parameters |
| **Refresh Dynamic Table** | Refreshes table to make data available |

Also if the location of the node, node name, column level description, and table level description results in an `ALTER` statement, whereas other column or table level changes like data type change, column name change, column addition/deletion result in a `CREATE` statement.

### Changing Materialization Type From Latest Record Version Table to Transient Dimension Table

If the materialization type changes from dynamic table to transient dynamic table and there are changes in dynamic table config options or table level changes that result in ALTER, the following steps are executed:

1. **Clone Work node**
2. **Swap Work node**
3. **Drop Dynamic table**
4. **Alter Dynamic Table**
5. **Refresh Dynamic Table**
6. **Apply Table Clustering(if cluster key option is provided)**
7. **Resume Recluster Table(if cluster key option is provided)**

If the materialization type changes from dynamic table to transient dynamic table and there are changes in dynamic table config options or table level changes that result in `CREATE`, the following steps are executed:

1. **Drop dynamic table**
2. **Create Work dynamic transient table**
3. **Apply Table Clustering(if cluster key option is provided)**
4. **Resume Recluster Table(if cluster key option is provided)**

### Changing Materialization Type From Transient Dynamic Dimension Table to Latest Record Version

If the materialization type from transient dynamic table to dynamic table and there are changes in dynamic table config options, the following steps gets executed:

1. **Drop transient dimension table**
2. **Create Work dimension table**
3. **Apply Table Clustering(if cluster key option is provided)**
4. **Resume Recluster Table(if cluster key option is provided)**

### Recreating the Latest Record Version Table

If anything changes other than the configuration options specified in Altering the Latest Record Version Table, then the Dynamic Table will be recreated by running a `CREATE OR REPLACE`statement.

If the changes in node results in recreating the Dynamic table, then following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Drop table/transient table** | Table is dropped before recreating in case the node name or location is changed |
| **Create Work Dynamic table/Dynamic transient table**| Dynamic table is created|

### Redeploying a DAG of Latest Record Version

If an entire DAG of Dynamic Tables has been deployed and changes are made to a deployed Dynamic Table Coalesce will only redeploy Dynamic Tables that have changed metadata.

### Redeployment with no changes 

If the nodes are redeployed with no changes compared to previous deployment,then no stages are executed

### Latest Record Version Work Undeployment

A table will be dropped if all of these are true:

* The Dynamic Dimension Node is deleted from a Workspace.
* The Workspace is committed to Git.
* The Workspace committed to Git is deployed to a higher level environment.

| **Stage** | **Description** |
|-----------|----------------|
| **Drop Dynamic Table** | Removes table from target environment |

---

## Code

### Dynamic Table Work Code

* [Node definition](https://github.com/coalesceio/Dynamic-Table-Nodes/tree/main/nodeTypes/DynamicTableWork-347/definition.yml)
* [Create Template](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableWork-347/create.sql.j2)

### Dynamic Table Dimension Code

* [Node definition](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableDimension-350/definition.yml)
* [Create Template](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableDimension-350/create.sql.j2)

### Dynamic Table Latest Record Version Code

* [Node definition](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableLatestRecordVersion-351/definition.yml)
* [Create Template](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableLatestRecordVersion-351/create.sql.j2)

### Macros

* [Macros](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/macros/macro-1.yml)
