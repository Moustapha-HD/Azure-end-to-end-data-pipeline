# Azure end-to-end-data Pipeline

### Table of contents

### Overview
Implementation of an end-to-end pipeline to migrate an On-Premise database to Azure cloud. We will use these following services :
* Azure Data Factory to ingest the On-Prem SQL server database and create the end-to-end Pipeline.
*	Azure Data Lake Gen2 to store SQL database tables
*	Azure Databricks to transform data
*	Azure Synapse Analytics to load data into a database
*	Azure Active Directory to secure access to resources
*	Azure Key Vault to store and access secrets securely

### Architecture
<img width="1151" alt="Capture d’écran 2023-09-16 à 18 12 08" src="https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/149c9d12-758f-4028-a9d7-a447605829a5">

### Prerequisites
For this project, you will need:
1) Azure account
2) Azure Databricks
3) Minimal knowledge of Azure and Databricks, such as basic configuration of the different services.

### Phase I: On-Premise - SQL Server Management Studio (SSMS)

* AdventureWorks2022 database available [Here](https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2022.bak)
* We will only use the HumanRessources tables. To do this, use the “script1_lookup_all_tables” script available in the repo.

![Sans titre](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/9119fa80-be0c-4058-92fc-15e315618da5)


### Phase II: Data ingestion on Azure Data Factory
ADF provides a cloud-based ETL solution that orchestrates data movement by scheduling data pipelines and transforming data at scale between various data stores and compute resources.

![1](https://github.com/Moustapha-HD/Azure-end-to-end-data-pipeline/assets/118195267/68334873-09f4-4459-a6d4-dfcdcd95bea9)

#### Lookup activity:
Microsoft documentation on the Lookup activity available [Here](https://learn.microsoft.com/fr-fr/azure/data-factory/control-flow-lookup-activity)

##### Source dataset
* Create a “linked service” to connect to the SQL server. Requires a Self Hosted Integration Runtime since we are trying to connect to an on-premise server.
* For security reasons, it is best to store passwords in Azure key vault.
* Query: See “script1_lookup_all_tables”

#### ForEach activity
