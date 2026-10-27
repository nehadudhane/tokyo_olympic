# Azure Data Factory ARM exports

This directory contains three groups of ARM deployment files for the Tokyo Olympics data ingestion pipeline.

- `factory/`: Data Factory resource template and parameters. The exported system-assigned managed identity `principalId` and `tenantId` have been removed from the template because these values are deployment-specific.
- `standalone/`: Full pipeline template containing linked services, datasets, and the `data-ingestion` pipeline, plus its parameters.
- `linked-deployment/`: Alternative linked deployment: `ArmTemplate_master.json` references `ArmTemplate_0.json`, along with its parameter file.

## Security and deployment notes

- No storage account keys or SAS tokens are included as values in these parameter files. Do not commit populated key or token parameters.
- The pipeline uses an Azure Data Lake Storage Gen2 account-key linked service, which needs a credential provided securely at deployment.
- The linked deployment requires an accessible templates container URI and SAS token.
- The original environment-specific factory and storage URL defaults remain unchanged; update them for your Azure environment before deployment.
- Deployment and successful pipeline execution have **not** been verified from these exported files.

These files document the ADF ingestion layer only; they do not include Azure Databricks, Synapse or Power BI artifacts.
