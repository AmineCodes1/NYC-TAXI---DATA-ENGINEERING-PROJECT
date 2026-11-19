# NYC Taxi Data Engineering Project

An end-to-end data engineering project that ingests 2025 NYC Green Taxi trip data, transforms it with Azure Databricks, and stores curated Delta Lake tables for analytics.

## Architecture

```text
NYC Taxi API (Parquet)
        |
        v
Azure Data Factory
  ForEachMonth (1-12)
        |
        v
ADLS Gen2 - Bronze
        |
        v
Azure Databricks / PySpark
  Silver notebook
        |
        v
ADLS Gen2 - Silver
        |
        v
Azure Databricks / Delta Lake
  Gold notebook
        |
        v
ADLS Gen2 - Gold + Databricks SQL tables
```

## What the pipeline does

1. Iterates over months `1` through `12` in the Azure Data Factory pipeline `ingestion-silver-gold`.
2. Downloads the corresponding Green Taxi Parquet file from the NYC Taxi data endpoint:
   `trip-data/green_tripdata_2025-MM.parquet`.
3. Writes the raw files to the ADLS Gen2 `bronze` filesystem under `trips_2025_data`.
4. Reads the raw trip data and reference files in the Silver notebook.
5. Cleans and reshapes the data:
   - Renames the trip type description column.
   - Splits taxi zone values into separate zone fields.
   - Derives trip date, year, and month fields.
   - Selects the core trip and fare columns.
6. Writes transformed Parquet data to the `silver` filesystem.
7. Loads Silver data into Delta Lake tables in the `gold` database.
8. Demonstrates Delta Lake updates, deletes, history, and time travel.

## Repository structure

```text
.
├── databricks/
│   ├── Silver_Notebook.ipynb       # Bronze-to-Silver transformations
│   └── Gold_Notebook.ipynb         # Silver-to-Gold Delta tables and examples
├── dataset/
│   ├── nyc_api_dataset.json        # Parameterized NYC API Parquet dataset
│   └── nyc_raw_sink.json           # ADLS Bronze Parquet sink
├── factory/
│   └── nyctaxiadfamine.json        # Azure Data Factory definition
├── linkedService/
│   ├── NYC_WEB_API.json             # NYC Taxi HTTP source
│   ├── datalakestorage.json         # ADLS Gen2 connection
│   └── nyc_db.json                  # Azure Databricks connection
├── pipeline/
│   └── ingestion-silver-gold.json   # Orchestration pipeline
└── publish_config.json              # Data Factory publish metadata
```

## Azure resources

The exported configuration references:

- Azure Data Factory for orchestration.
- Azure Data Lake Storage Gen2 with `bronze`, `silver`, and `gold` filesystems.
- Azure Databricks for PySpark processing and Delta Lake.
- The NYC Taxi public Parquet endpoint hosted behind CloudFront.

Resource names, subscriptions, workspace URLs, and credentials are environment-specific. Before deploying, replace them with values from your own Azure environment and configure authentication through managed identity or a secure secret store.

## Prerequisites

- An Azure subscription.
- Azure Data Factory.
- An ADLS Gen2 storage account with `bronze`, `silver`, and `gold` filesystems.
- An Azure Databricks workspace with a cluster or job capable of running PySpark and Delta Lake.
- Permissions for Data Factory and Databricks to read and write the storage account.
- Git, Azure CLI, or the Azure portal for deployment.

## Deployment outline

1. Create the ADLS Gen2 storage account and the `bronze`, `silver`, and `gold` filesystems.
2. Create an Azure Databricks workspace and configure access to ADLS Gen2.
3. Import the notebooks from `databricks/`.
4. Create or import the linked services from `linkedService/`.
5. Import the datasets from `dataset/`.
6. Import the pipeline from `pipeline/`.
7. Update storage URLs, workspace references, authentication, and any environment-specific values.
8. Run `ingestion-silver-gold` and monitor the copy and Databricks activities in Azure Data Factory.
9. Query the resulting tables from the Databricks SQL workspace:

```sql
SELECT * FROM gold.trip_zone;
SELECT * FROM gold.trip_type;
SELECT * FROM gold.trip_trip;
```

## Data layers

| Layer | Format | Purpose |
| --- | --- | --- |
| Bronze | Parquet | Raw monthly NYC Green Taxi files |
| Silver | Parquet | Cleaned and reduced trip, zone, and trip type data |
| Gold | Delta Lake | Managed analytical tables registered in the `gold` database |

## Notes

- The current pipeline targets the 2025 monthly Green Taxi files.
- The pipeline expects all twelve monthly source files to be available.
- The notebooks use append mode for Silver writes and create Gold tables when they do not already exist. Review write modes before rerunning in a production environment.
- Do not commit storage keys, access tokens, client secrets, or other credentials. Use managed identities, Azure Key Vault, or Data Factory secure linked-service configuration.

## License

No license is currently specified for this repository. Add a license before distributing or reusing the project.
