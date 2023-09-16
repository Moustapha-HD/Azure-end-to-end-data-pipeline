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
1 Azure account
2 Azure Databricks
3 Minimal knowledge of Azure and Databricks, such as basic configuration of the different services.
