# Azure Data Factory Pipeline: dynamicload

## Overview
This Azure Data Factory (ADF) pipeline named **dynamicload** is designed to dynamically copy data from a source (Azure Blob Storage) to a target (Azure SQL Database) based on metadata retrieved from an Azure SQL Database table. The pipeline involves two key activities:
1. **Lookup1** - Fetch metadata information from the database.
2. **ForEach1** - Iterate through the metadata and perform dynamic data copying for each dataset.

---

## Pipeline Activities

### 1. Lookup1 Activity
- **Description**: This activity fetches the metadata from the `test.metadata` table in an Azure SQL Database. The metadata contains the source file name (`srcfile`), target schema (`tgtschema`), target table name (`tgttblname`), and the delimiter used (`delimeter`).

- **SQL Query**:
  ```sql
  SELECT DISTINCT srcfile, tgtschema, tgttblname, delimeter 
  FROM test.metadata
