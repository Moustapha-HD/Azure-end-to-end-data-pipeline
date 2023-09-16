# Azure end-to-end-data Pipeline

### Table of contents

* [Overview](#Overview)
* [Architecture](#Architecture)
* [Prerequisites](#Prerequisites)
* [Step I : On-Premise - SQL Server Management Studio](#step1)
* [Step II: Data ingestion on Azure Data Factory](#step2)
* [Step III : Databriks](#step3)
* [Step IV : Load Data in Silver and Gold layer](#step4)
* [Step V: Azure Synapse Analytics](#step5)
* [References](#References)
* [Contact](#Contact)



<a name="Overview"></a>
## Overview
Implementation of an end-to-end pipeline to migrate an On-Premise database to Azure cloud. We will use these following services :
* Azure Data Factory to ingest the On-Prem SQL server database and create the end-to-end Pipeline.
*	Azure Data Lake Gen2 to store SQL database tables
*	Azure Databricks to transform data
*	Azure Synapse Analytics to load data into a database
*	Azure Active Directory to secure access to resources
*	Azure Key Vault to store and access secrets securely

<a name="Architecture"></a>
## Architecture
<img width="1151" alt="Capture d’écran 2023-09-16 à 18 12 08" src="https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/149c9d12-758f-4028-a9d7-a447605829a5">

<a name="Prerequisites"></a>
## Prerequisites
For this project, you will need:
1) Azure account
2) Azure Databricks

> [!NOTE]
> Minimal knowledge of Azure and Databricks, such as basic configuration of the different services.

<a name="step1"></a>
## Step I : On-Premise - SQL Server Management Studio

* AdventureWorks2022 database available [Here](https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2022.bak)
* We will only use the HumanRessources tables. To do this, use the “script1_lookup_all_tables” script available in the repo.

![Sans titre](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/9119fa80-be0c-4058-92fc-15e315618da5)

<a name="step2"></a>
## Step II : Data ingestion on Azure Data Factory
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

<a name="step3"></a>
## Step III : Databriks

Azure Databricks provides the latest versions of Apache Spark and allows you to seamlessly integrate with open source libraries. Spin up clusters and build quickly in a fully managed Apache Spark environment with the global scale and availability of Azure.

In this part, we will use the three Notebooks available in the repo. The treatments carried out in each Notebook:
1) Storage_mount: To mount the three containers available in the storage account: “bronze”, “silver” and “gold”
2) Bronze_silver: To transfer data from the bronze layer to the silver layer and perform a simple transformation of the date format from “datetime” to “date”
3) Silver_gold: To make transformations on the nomenclature of the column names of each table

<a name="step4"></a>
## Step IV : Load Data in Silver and Gold layer
![7](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/3e4dc6a2-e7b0-4f3f-b838-b2f177362385)

### Notebook activity
#### Databricks linked service
* Since this is a service within Azure, create a linked service of type AutoResolveIntegrationRuntime
* Settings: Notebook path in Databricks.

<img width="454" alt="8" src="https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/2dee09f9-62db-4454-824f-94a783188c7b">

* We will make the same manipulation for the last activity: silver_gold.

After execution of the entire pipeline, the data will be copied into the “Bronze” directory through the Lookup and ForEach activities, then into the “Silver” and finally “Gold” directories through the last two Notebook activities.

The next phase will consist of creating an SQL database in Azure Synapse analytics.

<a name="step5"></a>
## Step V: Azure Synapse Analytics

Azure Synapse Analytics is a limitless analytics service that brings together data integration, enterprise data warehousing and big data analytics.

![Capture d’écran 2023-09-16 à 23 47 04](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/15cc86cf-ced6-4ced-a1f2-3fcebfc5c2c1)

### Get MetaData activity
#### Dataset: 
* Create a linked service to connect to Azure Data Lake Storage Gen2

![11](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/8ad0044a-9af6-44dd-802c-15f4568a70a8)

### For Each activity :
<img width="395" alt="10" src="https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/d3bce97e-28df-4b2b-9229-27df67aab9db">

#### Items: 
`@activity('get tablenames').output.childItems`

### Creation of the database
Create a serverless database: db_gold to store all tables

![Capture d’écran 2023-09-16 à 23 41 57](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/42984129-ff4c-4588-b6d0-a5d22ae2966d)

#### Stored procedure in the ForEach activities

Run the following script: “script2_sp_CreateSQLServerlessView_gold” to create a stored procedure

![Capture d’écran 2023-09-16 à 23 48 54](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/5d52a0cd-6f64-4613-9b81-78ff7187d1bf)
 
##### Linked service
Linked service to the serverless db_gold
##### Stored procedure name
Select the one executed through the script "script2_sp_CreateSQLServerlessView_gold"

Now, when you run the pipeline, all the data will be loaded into the SQL database in Views.

![Capture d’écran 2023-09-16 à 23 43 50](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/737132d5-ed99-4bce-a65f-e14708369042)

Query the db_gold database in Azure Synapse.

![Capture d’écran 2023-09-16 à 23 54 51](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/4280521f-b6ed-4766-b23f-ce14ee5ead9e)
 
Perfect, everything works good :+1:

<a name="references"></a>
## References

<a name="contact"></a>
## Contact
