# Dynamic Tables Package

The Dynamic Tables Package includes:

* [Dynamic Table Work](#dynamic-table-work)
* [Dynamic Table Dimension](#dynamic-table-dimension)
* [Dynamic Table Latest Record Version](#dynamic-table-latest-record-version)
* [Code](#code)

# Dynamic Table Work

The Coalesce Dynamic Table Work UDN is a versatile node that allows you to develop and deploy a single Dynamic Table Work or a DAG of Dynamic Tables in Snowflake. 

[Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about) are a new table type offered by Snowflake that allow data teams to use SQL statements to declaratively define the results of data pipelines. Dynamic tables simplify the process of creating and managing data pipelines by streamlining data transformations without having to manage Streams and Tasks. 

## Version Information
 * UDN Name: Dynamic Table Work
 * Package Name: 

 * Certified Version: 1.1
 * Documentation Version: 1.1
 * Current Version: 1.1
 * Originally Developed by: John Gontarz
 * Current Owner: John Gontarz
 * Materialization Type: Dynamic Table/transient dynamic table
 * Create Template: Yes
 * Run Template: No

 * Deployment Strategy: Advanced
 * Deployment Method: ALTER when possible otherwise CREATE OR REPLACE

 * Test Enabled: No
   
## Node Configuration
The Dynamic Table Dimension has three configuration groups:

* [Node Properties](#node-properties)
* [Dynamic Table Options](#dynamic-table-options)
* [General Options](#general-options)

![node_config_options](https://github.com/coalesceio/Dynamic-Table-Nodes/assets/7216836/2be55d50-4ef6-4acc-a658-c14edb234822)

<img src="https://github.com/Coalesce-Software-Inc/node_marketplace/blob/75a94c9ac5b8aac1c1faf8a6948fbd024c9ef2e9/images/node_config_options.png" alt="Dynamic Tables Dimension Node Configuration Options" style="width:280px;height:150px;">

### Node Properties
There are four configs within the **Node Properties** group.

* **Storage Location(required)**: Storage Location where the Dynamic Table will be created.
* **Node Type(required)**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled(required)**: 
  * If TRUE the node will be deployed or redeployed when changes are detected.
  * If FALSE the node will not be deployed or the node will be dropped during redeployment.

### Dynamic Table Options

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

<img src="https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/bad33b2fa51c18a63854ae167d6a983b6cf491f1/images/Dynamic_Table_Options.png" alt="Dynamic Tables Dimension Node Configuration Options" style="width:280px;height:400px;">
  
### General Options

There are three configs related to the General Options group.

* **Distinct**: True or False toggle that specifies whether or not to return DISTINCT rows.
* **Group By All**: True or False toggle that specifies whether or not to add non-aggregated columns to GROUP BY.
* **Multi Source**: True or False toggle that specifies whether or not multiple sources will be combined using either UNION or UNION ALL.
* **Create As**:Provides option to choose materialization type as ‘dynamic table’ or ‘transient dynamic table’
* **Cluster key**: True/False to determine whether Dimension is to be clustered or not
    * True -Allows you to specify the column based on which clustering is to be done.
            -Allow Expressions Cluster Key-True ->allows to add an expression to the specified cluster key
    * False – No clustering done

<img src="https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/aafb753b053abae7e775da246823b4cda533912a/images/dynamic%20table1.1_general.png" alt="Dynamic Tables Dimension Node Configuration Options" style="width:280px;height:350px;">

## Deployment

### Deployment Parameters
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

### Initial Deployment
When deployed for the first time into an environment the Dynamic Table Work node will execute the following stage:

**Create Dynamic Work Table**
This stage will execute a `CREATE OR REPLACE` statement and create a Dynamic Table in the target environment.

### Deploying a DAG of Dynamic Tables

When a DAG of related Dynamic Tables are deployed together Coalesce will deploy the Dynamic Tables in the order that the Dynamic Tables are ordered.


## Redeployment

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

### Changing materialization type and dynamic table config options

#### Changing  materialization type from dynamic table to transient dynamic table

If the materialization type from dynamic table to transient dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed

* **Clone Work node**
* **Swap Work node**
* **Drop dynamic table**
* **Alter Dynamic Table**
* **Refresh Dynamic Table**
* **Apply Table Clustering(if cluster key option is provided)**
* **Resume Recluster Table(if cluster key option is provided)**

#### Changing  materialization type from transient dynamic table to dynamic table

If the materialization type from transient dynamic table to dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed

* **Drop transient dynamic table**
* **Create dynamic table**
* **Alter Dynamic Table**
* **Refresh Dynamic Table**
* **Apply Table Clustering(if cluster key option is provided)**
* **Resume Recluster Table(if cluster key option is provided)**

### Recreating the Dynamic Table

If anything changes other than the configuration options specified in [Altering the Dynamic Table](#altering-the-dynamic-table) then the Dynamic Table will be recreated by running a `CREATE OR REPLACE `statement.

### Redeploying a DAG of Dynamic Tables
If an entire DAG of Dynamic Tables has been deployed and changes are made to a deployed Dynamic Table Coalesce will only redeploy Dynamic Tables that have changed metadata.

## Undeployment

A table will be dropped if all of these are true:

* The Dynamic Work Node is deleted from a Workspace.
* The Workspace is committed to Git. 
* The Workspace committed to Git is deployed to a higher level environment.

This is executed as a single stage:

`Drop Dynamic Table`

# Dynamic Table Dimension

The Coalesce Dynamic Table Dimension UDN is a versatile node that allows you to develop and deploy a single Dynamic Table Dimension or a DAG of Dynamic Tables in Snowflake. 

[Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about. ) are a new table type offered by Snowflake that allow data teams to use SQL statements to declaratively define the results of data pipelines. Dynamic tables simplify the process of creating and managing data pipelines by streamlining data transformations without having to manage Streams and Tasks. 

## Version Information
 * UDN Name: Dynamic Table Dimension
 * Package Name: 

 * Certified Version: 1.1
 * Documentation Version: 1.1
 * Current Version: 1.1
 * Originally Developed by: John Gontarz
 * Current Owner: John Gontarz
 * Materialization Type: Dynamic Table/transient dynamic table
 * Create Template: Yes
 * Run Template: No

 * Deployment Strategy: Advanced
 * Deployment Method: ALTER when possible otherwise CREATE OR REPLACE

 * Test Enabled: No

## Node Configuration
The Dynamic Table Dimension has three configuration groups:

* [Node Properties](#node-properties)
* [Dynamic Table Options](#dynamic-table-options)
* [Dimension Options](#dimension-options)
  
<img src="https://github.com/Coalesce-Software-Inc/node_marketplace/blob/8b9d65fc3d60fe551671c404778c766d797305d0/images/dynamic_table_dimension_options.png" style="width:280px;height:150px;" alt="Dynamic table dimension configuration options">

### Dimension Options

There are three configs within the Dimension group.

* **Table keys (required):** The business keys columns based on which the Dimension key is formed.
* **Record versioning (required):** Type of column based on which columns that maintain history are updated.
  * Datetime column
  * Date and Time column
* **Timestamp or sequence:** The timestamp column name needs to be specified if Datetime column is chosen for Record versioning.
* **Date/Timestamp Columns:** Date column,time column, and sort order of the columns to be specified if Date column and Time column is chosen for Record versioning.

<img src="https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/ad212918d499922dd35cc71dd52f8c0ffb93164d/images/Dynamic%20table%20dimension%20options.png" alt="Dynamic Tables Dimension Node Configuration Options" style="width:280px;height:400px;">

### General Options

There are two configs related to the General Options group.

* **Create As**:Provides option to choose materialization type as ‘dynamic table’ or ‘transient dynamic table’
* **Cluster key**: True/False to determine whether Dimension is to be clustered or not
    * True -Allows you to specify the column based on which clustering is to be done.
            -Allow Expressions Cluster Key-True ->allows to add an expression to the specified cluster key
    * False – No clustering done
 
<img src="https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/aafb753b053abae7e775da246823b4cda533912a/images/dynamictable1.1-dimgeneral.png" alt="Dynamic Tables Dimension Node Configuration Options" style="width:280px;height:350px;">

## Deployment

### Deployment Parameters

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

### Initial Deployment

When deployed for the first time into an environment the Dynamic Table Latest Record Version node will execute the below stage:

**Create Dynamic Work Table**

This stage will execute a `CREATE OR REPLACE` statement and create a Dynamic Table in the target environment.

#### Deploying a DAG of Dynamic Tables

When a DAG of related Dynamic Tables are deployed together Coalesce will deploy the Dynamic Tables in the order that the Dynamic Tables are ordered.

### Redeployment

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
  
### Changing materialization type and dynamic table config options

#### Changing  materialization type from dynamic table to transient dynamic table

If the materialization type from dynamic table to transient dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed

* **Clone Dimension node**
* **Swap Dimension node**
* **Drop dynamic table**
* **Alter Dynamic Table**
* **Refresh Dynamic Table**
* **Apply Table Clustering(if cluster key option is provided)**
* **Resume Recluster Table(if cluster key option is provided)**

#### Changing  materialization type from transient dynamic table to dynamic table

If the materialization type from transient dynamic table to dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed

* **Drop transient dynamic table**
* **Create dimension dynamic table**
* **Alter Dynamic Table**
* **Refresh Dynamic Table**
* **Apply Table Clustering(if cluster key option is provided)**
* **Resume Recluster Table(if cluster key option is provided)**

#### Recreating the Dynamic Table

If anything changes other than the configuration options specified in [Altering the Dynamic Table](#altering-the-dynamic-table-1) then the Dynamic Table will be recreated by running a `CREATE OR REPLACE` statement.

#### Redeploying a DAG of Dynamic Tables

If an entire DAG of Dynamic Tables has been deployed and changes are made to a deployed Dynamic Table Coalesce will only redeploy Dynamic Tables that have changed metadata.

### Undeployment

A table will be dropped if all of these are true:

* The Dynamic Table Latest Record Version Node is deleted from a Workspace.
* The Workspace is committed to Git. 
* The Workspace committed to Git is deployed to a higher level environment.

This is executed as a single stage:

`Drop Dynamic Table`

# Dynamic Table Latest Record Version

The Coalesce Dynamic Table Latest Record Version UDN is a versatile node that allows you to develop and deploy a single Dynamic Table Work or a DAG of Dynamic Tables with only the latest version of rows in Snowflake.

[Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about. ) are a new table type offered by Snowflake that allow data teams to use SQL statements to declaratively define the results of data pipelines. Dynamic tables simplify the process of creating and managing data pipelines by streamlining data transformations without having to manage Streams and Tasks. 
  
## Version Information
 * UDN Name: Dynamic Table Latest Record Version
 * Package Name: 

 * Certified Version: 1.1
 * Documentation Version: 1.1
 * Current Version: 1.1
 * Originally Developed by: John Gontarz
 * Current Owner: John Gontarz
 * Materialization Type: Dynamic Table/transient dynamic table
 * Create Template: Yes
 * Run Template: No

 * Deployment Strategy: Advanced
 * Deployment Method: ALTER when possible otherwise CREATE OR REPLACE

 * Test Enabled: No

## Node Configuration
The Dynamic Table Latest Record Version has three configuration groups:

* [Node Properties](#node-properties)
* [Dynamic Table Options](#dynamic-table-options)
* [Record Versioning Options](#record-versioning-options)

### DAG of Dynamic tables
When designing DAG of Dynamic tables,it is ideal to specify the target lag as explained in the link below

[DAG of Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-refresh)

### Record Versioning Options

There are three configs within the Record Versioning Options group.

* **Table keys (required):** The business keys columns based on which the Dimension key is formed.
* **Record versioning (required):** Type of column based on which columns that maintain history are updated.
  * Datetime column
  * Date and Time column
* **Timestamp or sequence:** The timestamp column name needs to be specified if Datetime column is chosen for Record versioning.
* **Date/Timestamp Columns:** Date column,time column, and sort order of the columns to be specified if Date column and Time column is chosen for Record versioning.

<img src="https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/ad212918d499922dd35cc71dd52f8c0ffb93164d/images/Record%20versioning.png" alt="Dynamic Tables Dimension Node Configuration Options" style="width:280px;height:400px;">

### General Options

There are two configs related to the General Options group.

* **Create As**:Provides option to choose materialization type as ‘dynamic table’ or ‘transient dynamic table’
* **Cluster key**: True/False to determine whether Dimension is to be clustered or not
    * True -Allows you to specify the column based on which clustering is to be done.
            -Allow Expressions Cluster Key-True ->allows to add an expression to the specified cluster key
    * False – No clustering done
 
<img src="https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/aafb753b053abae7e775da246823b4cda533912a/images/dynamictable1.1-dimgeneral.png" alt="Dynamic Tables Dimension Node Configuration Options" style="width:280px;height:350px;">
## Deployment

### Deployment Parameters

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

### Initial Deployment

When deployed for the first time into an environment the Dynamic Table Latest Record Version node will execute the below stage:

**Create Dynamic Work Table**

This stage will execute a `CREATE OR REPLACE` statement and create a Dynamic Table in the target environment.

#### Deploying a DAG of Dynamic Tables

When a DAG of related Dynamic Tables are deployed together Coalesce will deploy the Dynamic Tables in the order that the Dynamic Tables are ordered.

### Redeployment

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
  
### Changing materialization type and dynamic table config options

#### Changing  materialization type from dynamic table to transient dynamic table

If the materialization type from dynamic table to transient dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed

* **Clone Work node**
* **Swap Work node**
* **Drop dynamic table**
* **Alter Dynamic Table**
* **Refresh Dynamic Table**
* **Apply Table Clustering(if cluster key option is provided)**
* **Resume Recluster Table(if cluster key option is provided)**

#### Changing  materialization type from transient dynamic table to dynamic table

If the materialization type from transient dynamic table to dynamic table and also if there are changes in dynamic table config options ,the following steps gets executed

* **Drop transient dynamic table**
* **Create dynamic table**
* **Alter Dynamic Table**
* **Refresh Dynamic Table**
* **Apply Table Clustering(if cluster key option is provided)**
* **Resume Recluster Table(if cluster key option is provided)**

#### Recreating the Dynamic Table

If anything changes other than the configuration options specified in [Altering the Dynamic Table](#altering-the-dynamic-table-1) then the Dynamic Table will be recreated by running a `CREATE OR REPLACE` statement.

#### Redeploying a DAG of Dynamic Tables

If an entire DAG of Dynamic Tables has been deployed and changes are made to a deployed Dynamic Table Coalesce will only redeploy Dynamic Tables that have changed metadata.

### Undeployment

A table will be dropped if all of these are true:

* The Dynamic Table Latest Record Version Node is deleted from a Workspace.
* The Workspace is committed to Git. 
* The Workspace committed to Git is deployed to a higher level environment.

This is executed as a single stage:

`Drop Dynamic Table`

# Code

## Dynamic table work
* [Node definition](https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/ee64505006f9786db1081f55aeebaab58c3acb99/Code-files/Dynamic%20Table%20Work%201.1-Node%20def.yml)
* [Create Template](https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/888ee2adf171cd339c7797310dd2c3aa59f656a9/Code-files/Dynamic%20worke1.1-create1.js.sql)

## Dynamic table dimension
* [Node definition](https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/ee64505006f9786db1081f55aeebaab58c3acb99/Code-files/Dynamic%20table%20dimension1.1-node%20def.yml)
* [Create Template](https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/888ee2adf171cd339c7797310dd2c3aa59f656a9/Code-files/Dynamicdimension1.1-create1.js.sql)

## Dynamic table latest record version

* [Node definition](https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/ee64505006f9786db1081f55aeebaab58c3acb99/Code-files/Dynamic%20table%20latest%20record%20version1.1-node%20def.yml)
* [Create Template](https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/888ee2adf171cd339c7797310dd2c3aa59f656a9/Code-files/Dynamiclv-create1.js.sql)
  
  [Macros](https://github.com/Coalesce-Software-Inc/coalesce_marketplace/blob/28da7344409986af70169700d8fcb73749ed8afe/Code-files/Macros_dynamictable.txt)
