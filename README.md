# Dynamic Tables Package

The Dynamic Tables Package includes:

* [Dynamic Table Work](#dynamic-table-work)
* [Dynamic Table Dimension](#dynamic-table-dimension)
* [Dynamic Table Latest Record Version](#dynamic-tables-latest-record-version)
* [Code](#code)

<h2 id="dynamic-table-work">Dynamic Table Work</h2>

The Coalesce Dynamic Table Work UDN is a versatile node that allows you to develop and deploy a single Dynamic Table Work or a DAG of Dynamic Tables in Snowflake.

[Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about) are a new table type offered by Snowflake that allow data teams to use SQL statements to declaratively define the results of data pipelines. Dynamic tables simplify the process of creating and managing data pipelines by streamlining data transformations without having to manage Streams and Tasks. 


### Dynamic Table Work Node Configuration

The Dynamic Table Dimension has three configuration groups:

* [Node Properties](#dynamic-table-work-node-properties)
* [Dynamic Table Options](#dynamic-table-options)
* [General Options](#dynamic-table-general-options)

Go to the node and select the **Config tab** to see the Node Properties, Dynamic Table Options and General Options.

<h3 id="dynamic-table-work-node-properties">Dynamic Table Work Node Properties</h3>

There are four configs within the **Node Properties** group.

* **Storage Location(required)**: Storage Location where the Dynamic Table will be created.
* **Node Type(required)**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled(required)**:
  * If TRUE the node will be deployed or redeployed when changes are detected.
  * If FALSE the node will not be deployed or the node will be dropped during redeployment.

<h3 id="dynamic-table-options">Dynamic Table Options</h3>

There are three configs within the **Dynamic Table Options** group.

* **Warehouse on which to execute Dynamic Table (required)**: The name of the warehouse used to refresh the Dynamic Table.
* **Downstream(required)**: True or False Toggle to determine Dynamic Table refresh schedule.
  * True - Specifies that the dynamic table should be refreshed on demand when other dynamic tables that depend on it need to refresh.
  * False - Allows you to set a Lag Specification for the Dynamic Table refresh.
* **Lag Specification**:  If the Downstream option is set to false you can set the refresh schedule using Lag Specification. Review [Snowflakes Dynamic Tables Refresh](https://docs.snowflake.com/en/user-guide/dynamic-tables-refresh) to understand how to specify the target lag.
  * **Time Value**: Number representing the frequency of refresh for a given Time Period.
  * **Time Period**: Seconds / Minutes / Hours Days related to the entered Time Value.
* **Refresh_Mode(required)**: Specifies the refresh type for the dynamic table.
  * **AUTO**: Enforces an incremental refresh of the dynamic table by default. If the CREATE DYNAMIC TABLE statement does not support the incremental refresh mode, the dynamic table is automatically created with the full refresh mode.
  * **INCREMENTAL**: Enforces an incremental refresh of the dynamic table.
  * **FULL**: Enforces a full refresh of the dynamic table
* **Initialize(required)**: Specifies the behavior of the initial refresh of the dynamic table. 
  * **ON_CREATE**: Refreshes the dynamic table synchronously at creation
  * **ON_SCHEDULE**: Refreshes the dynamic table at the next scheduled refresh.
    
 ![dynamic table options](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/169126315/e4fbc362-f0c6-463c-9e82-ac454709c700)

<h3 id="dynamic-table-general-options">General Options</h3>

There are three configs related to the General Options group.

* **Distinct**: True or False toggle that specifies whether or not to return DISTINCT rows.
* **Group By All**: True or False toggle that specifies whether or not to add non-aggregated columns to GROUP BY.
* **Multi Source**: True or False toggle that specifies whether or not multiple sources will be combined using either UNION or UNION ALL.
* **Create As**:Provides option to choose materialization type as ‘dynamic table’ or ‘transient dynamic table’
* **Cluster key**: True/False to determine whether Dimension is to be clustered or not
  * True -Allows you to specify the column based on which clustering is to be done.
            -Allow Expressions Cluster Key-True ->allows to add an expression to the specified cluster key
  * False – No clustering done

![dynamic table1 1_general](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/7216836/2c663f13-8e7c-43c0-a7e0-c4abc1edfa14)

## Dynamic Table Deployment

### Dynamic Table Deployment Parameters

The Dynamic Table Work includes an environment parameter that allows you to specify a different warehouse used to refresh a Dynamic Table in different environments.

The parameter name is `targetDynamicTableWarehouse` and the default value is `DEV ENVIRONMENT`.

When set to `DEV ENVIRONMENT` the value entered in the Dynamic Table Options config **Warehouse on which to execute Dynamic Table** will be used when creating the Dynamic Table.

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

### Dynamic Table Initial Deployment

When deployed for the first time into an environment the Dynamic Table Work node will execute the following stage:

* **Create Dynamic Work Table**: This stage will execute a `CREATE OR REPLACE` statement and create a Dynamic Table in the target environment.

### Deploying a DAG of Dynamic Tables

When a DAG of related Dynamic Tables are deployed together Coalesce will deploy the Dynamic Tables in the order that the Dynamic Tables are ordered.

### Dynamic Table Redeployment


After the Dynamic Table Work has deployed for the first time into a target environment, subsequent deployments may result in either altering the Dynamic Table or recreating the Dynamic table.

If a Dynamic Table is altered this will run two stages:

* **Stage 1: Alter Dynamic Table**
  * This stage will execute an `ALTER` statement and alter the Dynamic Table in the target environment setting the new parameters.
* **Stage 2: Refresh Dynamic Table**
  * This stage will refresh the Dynamic Table so that data is available after refresh is complete.

### Altering the Dynamic Table

There are three config changes that if made in isolation or all-together will result in an `ALTER` statement to modify the Dynamic Table in the target environment.

* Warehouse Name
* Downstream
* Lag Specification

### Changing Materialization Type and Dynamic Table Config Options

#### Changing  Materialization Type From Dynamic Table to Transient Dynamic Table

If the materialization type from dynamic table to transient dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed:

1. **Clone Work node**
2. **Swap Work node**
3. **Drop dynamic table**
4. **Alter Dynamic Table**
5. **Refresh Dynamic Table**
6. **Apply Table Clustering(if cluster key option is provided)**
7. **Resume Recluster Table(if cluster key option is provided)**

#### Changing  Materialization Type From Transient Dynamic Table to Dynamic Table

If the materialization type from transient dynamic table to dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed:

1. **Drop transient dynamic table**
2. **Create dynamic table**
3. **Alter Dynamic Table**
4. **Refresh Dynamic Table**
5. **Apply Table Clustering(if cluster key option is provided)**
6. **Resume Recluster Table(if cluster key option is provided)**

### Recreating the Dynamic Table

If anything changes other than the configuration options specified in [Altering the Dynamic Table](#altering-the-dynamic-table) then the Dynamic Table will be recreated by running a `CREATE OR REPLACE `statement.

### Redeploying a DAG of Dynamic Tables

If an entire DAG of Dynamic Tables has been deployed and changes are made to a deployed Dynamic Table Coalesce will only redeploy Dynamic Tables that have changed metadata.

### Dynamic Tables Undeployment

A table will be dropped if all of these are true:

* The Dynamic Work Node is deleted from a Workspace.
* The Workspace is committed to Git. 
* The Workspace committed to Git is deployed to a higher level environment.

This is executed as a single stage:

`Drop Dynamic Table`

<h2 id="dynamic-table-dimension">Dynamic Table Dimension</h2>

The Coalesce Dynamic Table Dimension UDN is a versatile node that allows you to develop and deploy a single Dynamic Table Dimension or a DAG of Dynamic Tables in Snowflake.

[Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about) are a new table type offered by Snowflake that allow data teams to use SQL statements to declaratively define the results of data pipelines. Dynamic tables simplify the process of creating and managing data pipelines by streamlining data transformations without having to manage Streams and Tasks.

### Dynamic Table Dimension Node Configuration

The Dynamic Table Dimension has three configuration groups:

* [Node Properties](#dynamic-table-dimension-node-properties)
* [Dynamic Table Options](#dynamic-table-dimension-options)
* [Dimension Options](#dimension-options)
* [General Options](#general-options)


Go to the node and select the **Config tab** to see the Node Properties, Dynamic Table Options and General Options.
  
<h3 id="dynamic-table-dimension-node-properties">Node Properties</h3>

* **Storage Location**: Storage Location where the WORK will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h3 id="dynamic-table-dimension-options"> Dynamic Table Options</h3>

There are three configs within the **Dynamic Table Options** group.

* **Warehouse on which to execute Dynamic Table (required)**: The name of the warehouse used to refresh the Dynamic Table.
* **Downstream(required)**: True or False Toggle to determine Dynamic Table refresh schedule.
  * True - Specifies that the dynamic table should be refreshed on demand when other dynamic tables that depend on it need to refresh.
  * False - Allows you to set a Lag Specification for the Dynamic Table refresh.
* **Lag Specification**:  If the Downstream option is set to false you can set the refresh schedule using Lag Specification. Review [Snowflakes Dynamic Tables Refresh](https://docs.snowflake.com/en/user-guide/dynamic-tables-refresh) to understand how to specify the target lag.
  * **Time Value**: Number representing the frequency of refresh for a given Time Period.
  * **Time Period**: Seconds / Minutes / Hours Days related to the entered Time Value.
* **Refresh_Mode(required)**: Specifies the refresh type for the dynamic table.
  * **AUTO**: Enforces an incremental refresh of the dynamic table by default. If the CREATE DYNAMIC TABLE statement does not support the incremental refresh mode, the dynamic table is automatically created with the full refresh mode.
  * **INCREMENTAL**: Enforces an incremental refresh of the dynamic table.
  * **FULL**: Enforces a full refresh of the dynamic table
* **Initialize(required)**: Specifies the behavior of the initial refresh of the dynamic table. 
  * **ON_CREATE**: Refreshes the dynamic table synchronously at creation
  * **ON_SCHEDULE**: Refreshes the dynamic table at the next scheduled refresh.

![dynamic table options](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/169126315/44235450-1383-4530-a68a-28f040bd5c48)

<h3 id="dynamic-table-dimension-options"> Dimension Options</h3>

There are three configs within the Dimension group.

* **Table keys (required):** The business keys columns based on which the Dimension key is formed.
* **Record versioning (required):** Type of column based on which columns that maintain history are updated.
  * Datetime column
  * Date and Time column
* **Timestamp or sequence:** The timestamp column name needs to be specified if Datetime column is chosen for Record versioning.
* **Date/Timestamp Columns:** Date column,time column, and sort order of the columns to be specified if Date column and Time column is chosen for Record versioning.

![Dimension options](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/169126315/b3f8d788-b8a8-4884-b9bd-2ff85039d9df)

<h3 id="general-dimension-options"> General Options </h3>

There are two configs related to the General Options group.

* **Create As**:Provides option to choose materialization type as ‘dynamic table’ or ‘transient dynamic table.
* **Cluster key**: True/False to determine whether Dimension is to be clustered or not.
	* True - Allows you to specify the column based on which clustering is to be done.
* **Allow Expressions Cluster Key**- True allows to add an expression to the specified cluster key.
  * False – No clustering done
 
![dynamictable1 1-dimgeneral](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/7216836/dda1c53c-a3da-4701-8449-0023db1ac44a)

## Dynamic Table Deployment

### Dynamic Table Deployment Parameters

The  Dynamic Table includes an environment parameter that allows you to specify a different warehouse used to refresh a Dynamic Table in different environments.

The parameter name is `targetDynamicTableWarehouse` and the default value is `DEV ENVIRONMENT`.

When set to `DEV ENVIRONMENT` the value entered in the Dynamic Table Options config Warehouse on which to execute Dynamic Table* will be used when creating the Dynamic Table.


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

### Dynamic Table Initial Deployment

When deployed for the first time into an environment the Dynamic Table Latest Record Version node will execute the below stage:

* **Create Dynamic Work Table**: This stage will execute a `CREATE OR REPLACE` statement and create a Dynamic Table in the target environment.


#### Deploying a DAG of Dynamic Tables

When a DAG of related Dynamic Tables are deployed together Coalesce will deploy the Dynamic Tables in the order that the Dynamic Tables are ordered.

### Dynamic Table Redeployment

After the Dynamic Table Latest Record Version has deployed for the first time into a target environment, subsequent deployments may result in either altering the Dynamic Table or recreating the Dynamic table.

If a Dynamic Table is to be altered this will run two stages:

* **Stage 1: Alter Dynamic Table**
  * This stage will execute an `ALTER` statement and alter the Dynamic Table in the target environment setting the new parameters.
* **Stage 2: Refresh Dynamic Table**
  * This stage will refresh the Dynamic Table so that data is available for analysis as soon as the refresh is complete.

#### Altering the Dynamic Table

There are three config changes that if made in isolation or all-together will result in an ALTER statement to modify the Dynamic Table in the target environment.

* Warehouse Name
* Downstream
* Lag Specification
  
### Changing Materialization Type and Dynamic Table Config Options

#### Changing  Materialization Type From Dynamic Table to Transient Dynamic Table

If the materialization type from dynamic table to transient dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed:

 1. **Clone Dimension node**
 2. **Swap Dimension node**
 3. **Drop dynamic table**
 4. **Alter Dynamic Table**
 5. **Refresh Dynamic Table**
 6. **Apply Table Clustering(if cluster key option is provided)**
 7. **Resume Recluster Table(if cluster key option is provided)**

#### Changing  Materialization Type From Transient Dynamic Table to Dynamic Table

If the materialization type from transient dynamic table to dynamic table and also if there are changes in dynamic table config options the following steps gets executed:

 1. **Drop transient dynamic table**
 2. **Create dimension dynamic table**
 3. **Alter Dynamic Table**
 4. **Refresh Dynamic Table**
 5. **Apply Table Clustering(if cluster key option is provided)**
 6. **Resume Recluster Table(if cluster key option is provided)**


#### Recreating the Dynamic Table

If anything changes other than the configuration options specified in [Altering the Dynamic Table](#altering-the-dynamic-table-1) then the Dynamic Table will be recreated by running a `CREATE OR REPLACE` statement.

#### Redeploying a DAG of Dynamic Tables

If an entire DAG of Dynamic Tables has been deployed and changes are made to a deployed Dynamic Table Coalesce will only redeploy Dynamic Tables that have changed metadata.

###  Dynamic Tables Undeployment

A table will be dropped if all of these are true:

* The Dynamic Table Latest Record Version Node is deleted from a Workspace.
* The Workspace is committed to Git. 
* The Workspace committed to Git is deployed to a higher level environment.

This is executed as a single stage:

* **Drop Dynamic Table**

<h2 id="dynamic-tables-latest-record-version">Dynamic Table Latest Record Version</h2>

The Coalesce Dynamic Table Latest Record Version UDN is a versatile node that allows you to develop and deploy a single Dynamic Table Work or a DAG of Dynamic Tables with only the latest version of rows in Snowflake.

[Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about. ) are a new table type offered by Snowflake that allow data teams to use SQL statements to declaratively define the results of data pipelines. Dynamic tables simplify the process of creating and managing data pipelines by streamlining data transformations without having to manage Streams and Tasks. 
  
### Dynamic Table Latest Record Version Node Configuration

The Dynamic Table Latest Record Version has three configuration groups:

* [Node Properties](#dynamic-table-latest-record-node-properties)
* [Record Versioning Options](#dynamic-table-latest-record-version-options)
* [General Options](#dynamic-table-latest-record-general-options)

Go to the node and select the **Config tab** to see the Node Properties, Dynamic Table Options and General Options.


<h4 id="dynamic-table-latest-record-node-properties">Node Properties</h4>


* **Storage Location**: Storage Location where the WORK will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h3 id="dynamic-table-latest-record-version-options">Dynamic Table Latest Record Version Record Versioning Options</h3>

There are three configs within the Record Versioning Options group.

* **Table keys (required):** The business keys columns based on which the Dimension key is formed.
* **Record versioning (required):** Type of column based on which columns that maintain history are updated.
  * Datetime column
  * Date and Time column
* **Timestamp or sequence:** The timestamp column name needs to be specified if Datetime column is chosen for Record versioning.
* **Date/Timestamp Columns:** Date column,time column, and sort order of the columns to be specified if Date column and Time column is chosen for Record versioning.

![Record versioning](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/7216836/f45665a6-0f91-4138-ad34-ba4d919f1e73)

<h3 id="dynamic-table-latest-record-general-options">Dynamic Table Latest Record Version General Options</h3>

There are two configs related to the General Options group.

* **Create As**:Provides option to choose materialization type as ‘dynamic table’ or ‘transient dynamic table.
* **Cluster key**: True/False to determine whether Dimension is to be clustered or not

  * True -Allows you to specify the column based on which clustering is to be done.

* **Allow Expressions Cluster Key**- True, allows to add an expression to the specified cluster key.
  * False – No clustering done.
 
![dynamictable1 1-dimgeneral](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/7216836/cfeb40cf-312b-4a5c-9f4f-11f1352d017e)


### DAG of Dynamic Table Latest Record Version

When designing DAG of Dynamic tables, you should specify the target lag. Review [# Understanding dynamic table refresh - Snowflake](https://docs.snowflake.com/en/user-guide/dynamic-tables-refresh)

### Dynamic Table Latest Record Version Deployment


#### Dynamic Table Latest Record Version Deployment Parameters

The  Dynamic Table includes an environment parameter that allows you to specify a different warehouse used to refresh a Dynamic Table in different environments.

The parameter name is `targetDynamicTableWarehouse` and the default value is `DEV ENVIRONMENT`.

When set to `DEV ENVIRONMENT` the value entered in the Dynamic Table Options config Warehouse on which to execute Dynamic Table* will be used when creating the Dynamic Table.


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

#### Dynamic Table Latest Record Version Initial Deployment

When deployed for the first time into an environment the Dynamic Table Latest Record Version node will execute the below stage:

* **Create Dynamic Work Table**: This stage will execute a `CREATE OR REPLACE` statement and create a Dynamic Table in the target environment.

#### Deploying a DAG of Dynamic Table Latest Record Version

When a DAG of related Dynamic Tables are deployed together Coalesce will deploy the Dynamic Tables in the order that the Dynamic Tables are ordered.

#### Dynamic Table Latest Record Version Redeployment

After the Dynamic Table Latest Record Version has deployed for the first time into a target environment, subsequent deployments may result in either altering the Dynamic Table or recreating the Dynamic table.

If a Dynamic Table is to be altered this will run two stages:

* **Stage 1: Alter Dynamic Table**
  * This stage will execute an `ALTER` statement and alter the Dynamic Table in the target environment setting the new parameters.
* **Stage 2: Refresh Dynamic Table**
  * This stage will refresh the Dynamic Table so that data is available for analysis as soon as the refresh is complete.

<h4 id="altering-dynamic-table-latest-record-version"> Altering the Dynamic Table Latest Record Version </h4>

There are three config changes that if made in isolation or all-together will result in an ALTER statement to modify the Dynamic Table in the target environment.

* Warehouse Name
* Downstream
* Lag Specification
  
### Changing Materialization Type and Dynamic Table Version Record Config Options


#### Changing  Materialization Type From Dynamic Table to Transient Dynamic Table Version Record


If the materialization type from dynamic table to transient dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed

 1. **Clone Work node**
 2. **Swap Work node**
 3. **Drop dynamic table**
 4. **Alter Dynamic Table**
 5. **Refresh Dynamic Table**
 6. **Apply Table Clustering(if cluster key option is provided)**
 7. **Resume Recluster Table(if cluster key option is provided)**


#### Changing  Materialization Type From Transient Dynamic Table to Dynamic Table Version Record


If the materialization type from transient dynamic table to dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed

1. **Drop transient dynamic table**
2. **Create dynamic table**
3. **Alter Dynamic Table**
4. **Refresh Dynamic Table**
5. **Apply Table Clustering(if cluster key option is provided)**
6. **Resume Recluster Table(if cluster key option is provided)**

#### Recreating the Dynamic Table Version Record

If anything changes other than the configuration options specified in [Altering the Dynamic Table](#altering-dynamic-table-latest-record-version) then the Dynamic Table will be recreated by running a `CREATE OR REPLACE` statement.

#### Redeploying a DAG of Dynamic Table Version Record

If an entire DAG of Dynamic Tables has been deployed and changes are made to a deployed Dynamic Table Coalesce will only redeploy Dynamic Tables that have changed metadata.

### Dynamic Table Latest Record Version Undeployment

A table will be dropped if all of these are true:

* The Dynamic Table Latest Record Version Node is deleted from a Workspace.
* The Workspace is committed to Git. 
* The Workspace committed to Git is deployed to a higher level environment.

This is executed as a single stage:


* **Drop Dynamic Table**

<h1 id="code">Code</h1>


## Dynamic Table Work

* [Node definition](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableWork-156/definition.yml)
* [Create Template](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableWork-156/create.sql.j2)

## Dynamic Table Dimension

* [Node definition](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableDimension-249/definition.yml)
* [Create Template](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableDimension-249/create.sql.j2)

## Dynamic Table Latest Record Version

* [Node definition](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableLatestRecordVersion-260/definition.yml)
* [Create Template](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/nodeTypes/DynamicTableLatestRecordVersion-260/create.sql.j2)

## Macros

* [Macros](https://github.com/coalesceio/Dynamic-Table-Nodes/blob/main/macros/macro-1.yml)


[<img src="https://github.com/coalesceio/Dynamic-Table-Nodes/assets/7216836/01631be6-0604-4e4f-9a64-825ec98eb3ea" alt="Git Logo" height="40">](https://github.com/coalesceio/Dynamic-Table-Nodes)
