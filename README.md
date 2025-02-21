# Azure Data Factory ETL Pipelines

This project contains a set of Azure Data Factory (ADF) pipelines designed for ETL (Extract, Transform, Load) processes. The pipelines are configured to copy data from various sources to Azure SQL Database.

## File Details

### adf_etl_pipeline_1.json
- **Description**: This pipeline copies data from a delimited text source to an Azure SQL table.
- **Activities**:
  - **Copy Data2**: Copies data from a delimited text source (`DelimitedText1`) to an Azure SQL table (`AzureSqlTable1`).

### adf_etl_pipelien2.json
- **Description**: This pipeline copies data from a binary source to an Azure SQL table.
- **Activities**:
  - **Copy Data1**: Copies data from a binary source (`Binary1`) to an Azure SQL table (`AzureSqlTable2`).

### adl_etl_pipeline4.json
- **Description**: This pipeline performs a lookup operation to retrieve metadata and then iterates over the results to copy data from Azure Blob Storage to Azure SQL Database.
- **Activities**:
  - **Lookup1**: Retrieves metadata from an Azure SQL table (`AzureSqlTable2`).
  - **ForEach1**: Iterates over the lookup results and copies data from Azure Blob Storage to Azure SQL Database.

### adf_etl_pipeline5.json
- **Description**: This pipeline copies data from a delimited text source stored in Azure Blob Storage to an Azure SQL table. It includes column mappings.
- **Activities**:
  - **Copy Data1**: Copies data from a delimited text source (`test1`) to an Azure SQL table (`AzureSqlTable2`). It maps specific columns from the source to the target.

### dynamic_multidelimeter.json
- **Description**: This pipeline dynamically handles multiple delimiters when copying data from Azure Blob Storage to Azure SQL Database.
- **Activities**:
  - **Lookup1**: Retrieves metadata from an Azure SQL table (`AzureSqlTable2`).
  - **ForEach1**: Iterates over the lookup results and copies data from Azure Blob Storage to Azure SQL Database, using the specified delimiter.

### dynamicload.json
- **Description**: This pipeline dynamically loads data from Azure Blob Storage to Azure SQL Database based on metadata.
- **Activities**:
  - **Lookup1**: Retrieves metadata from an Azure SQL table (`AzureSqlTable2`).
  - **ForEach1**: Iterates over the lookup results and copies data from Azure Blob Storage to Azure SQL Database.

## Overview - dynamicload
This Azure Data Factory (ADF) pipeline named **dynamicload** is designed to dynamically copy data from a source (Azure Blob Storage) to a target (Azure SQL Database) based on metadata retrieved from an Azure SQL Database table. The pipeline involves two key activities:
1. **Lookup1** - Fetch metadata information from the database.
2. **ForEach1** - Iterate through the metadata and perform dynamic data copying for each dataset.

## Pipeline Activities

### 1. Lookup1 Activity
- **Description**: This activity fetches the metadata from the `test.metadata` table in an Azure SQL Database. The metadata contains the source file name (`srcfile`), target schema (`tgtschema`), target table name (`tgttblname`), and the delimiter used (`delimeter`).

- **SQL Query**:
  ```sql
  SELECT DISTINCT srcfile, tgtschema, tgttblname, delimeter 
  FROM test.metadata
  ```

- **Properties**:
  - **Dataset**: `AzureSqlTable2` - Points to the Azure SQL Database.
  - **Output**: The result is stored as a collection of values used in the subsequent `ForEach` activity.
  - **First Row Only**: Set to `false`, meaning all rows from the query result will be returned.

### 2. ForEach1 Activity
- **Description**: This activity iterates over the collection of metadata fetched in the `Lookup1` activity and dynamically loads data from Azure Blob Storage to the Azure SQL Database.

- **Loop Behavior**:
  - The loop will process the items sequentially (`isSequential: true`).
  - The items for iteration are based on the expression:
    ```json
    @activity('Lookup1').output.value
    ```

#### a. Copy Data1 Activity (inside ForEach1)
- **Description**: The `Copy Data1` activity is responsible for copying data from a source file in Azure Blob Storage to the target table in the Azure SQL Database, truncating the target table before the load.

- **Source**:
  - **Type**: `DelimitedTextSource` - Reads delimited text files (e.g., CSV) from Azure Blob Storage.
  - **Settings**: 
    - Uses `AzureBlobStorageReadSettings` with recursive search enabled (`recursive: true`).
    - Reads the file specified in the metadata.

- **Sink**:
  - **Type**: `AzureSqlSink` - Writes the data to an Azure SQL Database table.
  - **Pre-Copy Script**: A truncate command is dynamically generated to clear the target table before loading new data:
    ```sql
    TRUNCATE TABLE [@{item().tgtschema}].[@{item().tgttblname}]
    ```
  
- **Inputs**:
  - **Dataset Reference**: `DS_test` - Dataset used to read from Azure Blob Storage, with the source file dynamically passed through the `SourceFile` parameter.
  
- **Outputs**:
  - **Dataset Reference**: `AzureDB1` - Points to the Azure SQL Database, with the schema and table name dynamically passed through the parameters (`tgtschema` and `tgttblname`).

---

## Configuration

### Datasets
1. **AzureSqlTable2**: Represents the Azure SQL Database where the metadata is stored.
2. **DS_test**: Represents the Azure Blob Storage containing the source data files.
3. **AzureDB1**: Represents the target Azure SQL Database where data is copied to.

### Parameters
- `SourceFile`: The file path in Azure Blob Storage is dynamically assigned using `@{item().srcfile}` from the metadata.
- `tgtschema` and `tgttblname`: The target schema and table name are dynamically assigned using `@{item().tgtschema}` and `@{item().tgttblname}` from the metadata.

---

## Error Handling
- The pipeline is set to retry failed activities with a timeout policy of **7 days** (`timeout: "7.00:00:00"`) and no retries (`retry: 0`).
  
---

## Notes
- Ensure the metadata in the `test.metadata` table is updated correctly before running the pipeline, as it drives the dynamic data loading process.
- The pre-copy script will truncate the target table before each load, so make sure that truncating the data is appropriate for your use case.

## Usage Instructions

1. **Prerequisites**:
   - Azure Data Factory instance.
   - Azure SQL Database.
   - Azure Blob Storage (if applicable).

2. **Setup**:
   - Deploy the pipelines to your Azure Data Factory instance.
   - Configure the linked services and datasets to point to your data sources and targets.

3. **Running the Pipelines**:
   - Trigger the pipelines manually or set up a schedule for automatic execution.

## Contributing
Contributions are welcome! Please fork the repository and submit a pull request with your changes.

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
