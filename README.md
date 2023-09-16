# Azure end-to-end-data Pipeline

## Table of contents

## Overview
Implementation of an end-to-end pipeline to migrate an On-Premise database to Azure cloud. We will use these following services :
* Azure Data Factory to ingest the On-Prem SQL server database and create the end-to-end Pipeline.
*	Azure Data Lake Gen2 to store SQL database tables
*	Azure Databricks to transform data
*	Azure Synapse Analytics to load data into a database
*	Azure Active Directory to secure access to resources
*	Azure Key Vault to store and access secrets securely

## Architecture
<img width="1151" alt="Capture d’écran 2023-09-16 à 18 12 08" src="https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/149c9d12-758f-4028-a9d7-a447605829a5">

## Prerequisites
For this project, you will need:
1) Azure account
2) Azure Databricks
3) Minimal knowledge of Azure and Databricks, such as basic configuration of the different services.

## Phase I: On-Premise - SQL Server Management Studio (SSMS)

* AdventureWorks2022 database available [Here](https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2022.bak)
* We will only use the HumanRessources tables. To do this, use the “script1_lookup_all_tables” script available in the repo.

![Sans titre](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/9119fa80-be0c-4058-92fc-15e315618da5)


## Phase II: Data ingestion on Azure Data Factory
ADF provides a cloud-based ETL solution that orchestrates data movement by scheduling data pipelines and transforming data at scale between various data stores and compute resources.

![1](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/68334873-09f4-4459-a6d4-dfcdcd95bea9)

### Lookup activity:
Microsoft documentation on the Lookup activity available [Here](https://learn.microsoft.com/fr-fr/azure/data-factory/control-flow-lookup-activity)

#### Source dataset
* Create a “linked service” to connect to the SQL server. Requires a Self Hosted Integration Runtime since we are trying to connect to an on-premise server.
* For security reasons, it is best to store passwords in Azure key vault.
* Query: See “script1_lookup_all_tables”

### ForEach activity
![2](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/aaf5847f-0671-451c-b817-db08b9f18f14)

Microsoft documentation on the ForEach activity available [Here](https://learn.microsoft.com/fr-fr/azure/data-factory/control-flow-for-each-activity)

#### Items: 
`@activity('Lookup all tables').output.value` : Allows you to retrieve the output of the lookup: The name of the schema (SchemaName) and the names of each table (TableName)

### Activities in ForEach:
![3](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/f0c438e7-25f7-4498-8faf-39d6a5407f05)

#### Source dataset: 
The same Integration Runtime as before.

#### Query
`@{concat('SELECT * FROM ', item().SchemaName, '.', item().TableName)}`: Allows you to query all tables as input to the forEach. Example: `SELECT * FROM HumanResources.Shift`

#### Sink
<img width="454" alt="4" src="https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/f467eb40-1862-4c6f-ad19-a066fbba6f10">

#### Sink dataset: 
* New => Connection
* Linked service: Linked service to connect to Azure Data Lake Storage Gen2
* File path:
    * First field: bronze #container to create in your storage account
    * Second field: `@{concat(dataset().schemaname, '/', dataset().tablename)}`
    * Third field: `@{concat(dataset().tablename,'.parquet')}`

#### Parameters
<img width="454" alt="5" src="https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/e8c2beb2-0463-4f39-8fdb-146f7ece69a9">

#### Dataset properties:
* schemaname: @item().SchemaName
* tablename: @item().TableName

After running the pipeline, all the tables will be copied to the “bronze” directory with the following nomenclature:
<img width="436" alt="5" src="https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/90fd4872-a616-4865-a0f2-fab92795e2e3">

## Phase III: Databriks

Azure Databricks provides the latest versions of Apache Spark and allows you to seamlessly integrate with open source libraries. Spin up clusters and build quickly in a fully managed Apache Spark environment with the global scale and availability of Azure.

For this part, we will use the three Notebooks available in the repo. The treatments carried out in each Notebook:
1) Storage_mount: To mount the three containers available in the storage account: “bronze”, “silver” and “gold”
2) Bronze_silver: To transfer data from the bronze layer to the silver layer and perform a simple transformation of the date format from “datetime” to “date”
3) Silver_gold: To make transformations on the nomenclature of the column names of each table

PHASE IV: Load Data in Silver and Gold layer with ADF
